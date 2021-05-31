---
layout: single
title: 'git restful: a thought experiment to help understand how git works, part 1'
excerpt: This is not an implementation of any kind but rather a thought experiment, which I found quite intuitive for ordinary developers to understand how git works and what happens behind different commands
date: 2021-05-31 22:48:12 +0800
---

This is not an implementation of any kind but rather a thought experiment, which I found quite intuitive for ordinary developers to understand how git works and what happens behind different commands. For sanity's sake, many details will be reduced to a simpler level or completely ignored.

Plus, an imaginary API for git sounds fun.

Assume we don't have a git implementation, and we're building a service to mimic one, our service consists of

- a server with data models and APIs to mutate the git repository
- a client to accept git commands and take actions

## server

### storage

Git uses a file-based data store. However, the underlying database is irrelevant to our imaginary service. All we need is a magical key-value store (an SQLite database, or multiple JSON files) with essential functions:

- to isolate values into different models
- to save a value as a specific model while generating a unique id (or use a provided one)
- to fetch a value by model and id

### the core models

Git itself is not that complex. It relies on only a trinity of three core models to operate. Additionally, there're references acting like annotated shortcuts.

#### blob

- a blob is simply a segment of binary content, without metadata like filename and timestamp
- it's a snapshot of a specific version of a file. After all, it's a version control service, and this is our tiniest unit to control
- we're using plain string to represent the content despite its binary nature

```ruby
id: id
content: string
```

#### tree

- a tree contains its path plus a list of (pointers of) other blobs or trees, just like a directory containing the path of itself plus files and sub-directories within
- we could recursively build a tree from only one root node TBD

```ruby
id: id
path: string
children: [{
  child_type: tree | blob
  child_id: id
}]
```

#### commit

- a commit is like a little yellow sticker with a piece of message attached to the trunk of a tree
- therefore, it consists of a message string and a pointer to a tree
- additionally, a commit may refer to one or more parent commits, collectively forming a network of different chains
    - a normal commit has one parent, and a merge commit has two. You may use `git merge` to create one with more than two parents, a.k.a an octopus merge

```ruby
id: id
tree_id: id
parent_ids: id[]
message: string
```

#### reference

- A reference is a named commit. That's it
- In reality, git doesn't treat references the same way as the other three core models, yet it doesn't matter to our fantasy service
- A **branch** is an ordinary reference, pointing to the head of a chain of commits. The id of a branch reference always formatted as `heads/{branch}`
- a **tag**, as you probably have guessed, is a reference as well, with an id like `tags/{tag}`
- HEAD is a particular local-only reference, marking your current position

```ruby
id: id
commit_id: id
```

#### to recap

- a blob is a snapshot of one file
- a tree is a snapshot of one directory
- a commit is a snapshot of the repository, with a piece of message
- a reference points to a commit that you often care about

### a sample scenario

We will define some preliminary data to represent a repository with exactly one file and one commit.

#### files

|path   |content|
|-------|-------|
|/constitution.md|We the People of…|

#### blobs

|id   |content          |
|-----|-----------------|
|blob1|We the People of…|

#### trees

|id     |path |children|
|-------|-----|--------|
|tree1  |/constitution.md|[]      |

#### commits

|id     |tree_id|message     |parent_ids|
|-------|-------|------------|----------|
|commit1|tree1  |first commit|[]        |

#### references

|id     |commit_id|
|-------|---------|
|heads/master|commit1  |
|HEAD   |commit1  |

## client

Following are the pseudocode snippets to handle different git commands.

### branch & checkout

The sweet command `git checkout` is a sweet shortcut for one `git branch` plus one `git checkout`. `git branch` creates the new branch, and `git checkout` updates the `HEAD` references.

```ruby
# git branch fix/amendment-1

current_head = GET /refs/HEAD
branch_name = 'fix/amendment-1'
branch_ref = POST /refs { id: 'heads/{branch_name}', commit_id: current_head.commit_id }

# git checkout fix/amendment-1

PATCH /refs/HEAD { commit_id: branch_ref.id }
```

### a lucid notation

The pseudocode above looks tedious and for better readability, let's try a painless notation:

- GET /models/id → Model.find
- POST /models → Model.create
- PATCH /models/id → Model.update
- …

### add & commit

We're ignoring concepts like local, remote, and staging, so adding and committing must be combined as an atomic action.

To add and commit files, we need to create new blobs for files, then build a new snapshot from modifying a clone of the current tree.

```ruby
# git add amendment-1.md && git commit 'fix: first amendment'

current_ref = Ref.find('heads/fix/amendment-1')
current_commit = Commit.find(current_ref.commit_id)
current_tree = Tree.find(current_commit.tree_id)

new_blob = Blob.create(content: 'Congress shall make…')
new_tree = Tree.create(path: 'amendment-1.md', children: [])
new_commit = Commit.create(
  tree_id: new_tree.id,
  message: 'fix: implemented first amendment',
  prev_commit_ids: [current_commit]
)

Ref.find('heads/fix/amendment-1').update(commit_id: new_commit.id)
Ref.find('HEAD').update(commit_id: new_commit.id)
```

### merge

Merging is troublesome, the variations of positions and amount of branches demand different actions, plus it may eventually fail due to conflicts.

We're mainly talking about merging branches right now. The mechanism of merging the actual content of files (blobs) is beyond the scope of this part of the post.

Merge is an operation perform against branches, but branches are only references pointing to commits, so technically, we're merging head commits of different chains.

The most straightforward case would be where the incoming commit is on the same chain, ahead of the base commit. In this case, we may simply move the pointer, a.k.a *fast-forwarding*, without creating a new commit.

```ruby
# git merge fix/amendment-1

   +--->---+
   |       |
---b---x---i

head = Ref.find('HEAD')
base_commit = Commit.find(head.commit_id)
incoming_commit = Ref.find('heads/fix/amendment-1')
base_commit.update(commit_id: incoming_commit.id)
```

This is often too good to be true. The branches (two or even more) to merge rarely rest on the same sub-chain, and we have to find one base commit, usually one of their common ancestors.

Finding that base commit involves different [strategies](https://git-scm.com/docs/merge-strategies), let's ease the headache and say we're merging only two branches and use the nearest common ancestor as the base. 

```ruby
# git merge fix/amendment-1

             (fix/amendment-1)
---c1---c2---x---|
---c4---x--------+
        (head)   (new commit)

head_ref = Ref.find('HEAD')
incoming_ref = Ref.find('heads/fix/amendment-1')
head_commit = Commit.find(head_ref).commit_id)
incoming_commit = Commit.find(incoming_ref.commit_id)

new_commit = Commit.create(
  tree_id: Tree.create(…).id,
  message: 'merge fix/amendment-1',
  prev_commit_ids: [head_commit.id, incoming_commit.id]
)

head_ref.update(commit_id: new_commit.id)
incoming_commit.update(commit_id: new_commit.id)
```

## next steps

### advanced merging and rebasing

We haven't touched the following topics:

- conflicts detection, merge strategy, octopus merge, etc.
- rebase command and the options like amend and squash

### branches as resources

A resource-oriented API service could be pretty flexible. One resource doesn't strictly bond with a corresponding model in the data store. In a friendly git service, for example, branches and tags could be defined as resources, providing functions like

- GET /branches?prefix='fix/'
- POST /branches?base=base

### conflicts and mergeability

Git hosting services tend to invent concepts like `pull/merge request` to handle mergeability, which is inevitably necessary for collaboration between committers. But for git itself, we could propose a more transparent suite of API like

- GET /conflicts?base=head&incoming=some-commit

### operation as a service

We could also design routes for operations like

- POST /operations/rebases?branch=some-branch

And now the status `409 Conflict` sounds perfectly appropriate.

Some would say this is not entirely `representational` since there's no resource created. I'd rather think of this as a shortcut for creating an ephemeral resource that has side effects.

## read more

- [https://git-scm.com/book/en/v2/Git-Internals-Git-Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
- [https://en.wikipedia.org/wiki/Git](https://en.wikipedia.org/wiki/Git#Data_structures)
- [https://docs.github.com/en/rest/reference/git](https://docs.github.com/en/rest/reference/git)
