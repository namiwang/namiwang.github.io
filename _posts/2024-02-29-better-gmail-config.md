---
layout: single
title: 'Managing Gmail Configuration for Labels and Filters as Code'
date: '2024-02-29 19:47:11 +0800'
classes: wide
---

> This post is based on my previous [thread](https://twitter.com/nami_m_wang/status/1759938276528656708), now further expanded with additional thoughts.

## Gmail searching/filtering issues

### confusing query builder

I feel like Gmail hasn't improved the searching/filtering part in years. It's a poor experience in general, especially for advanced usages.

For example, to filter emails by compound conditions, I must learn [a special syntax](https://support.google.com/mail/answer/7190?hl=en) and put the query in the input labeled "Has the words".

{% include figure.html
  src="2024-02-gmailctl/gmail-search-bad.jpeg"
  alt="gmail search experiences"
  caption="gmail search experiences"
  maxwidth="480px"
%}

In comparison, here is a common general-purpose query builder component ([react-querybuilder](https://github.com/react-querybuilder/react-querybuilder)). It covers all types of conditions here, providing accurate information and precise control. Why can't we have something like this, at least before inventing a query language?

{% include figure.html
  src="2024-02-gmailctl/react-querybuilder.png"
  alt="react-querybuilder"
  caption="react-querybuilder"
  maxwidth="480px"
%}

### bad importing experiences

Importing rules is just a disastrous experience.

- The embarrassing design feels like it's from the 90s: users have to click 3 times for "import rules", "choose file", and "open file".
- The modal auto closes itself after finishing, no matter succeeded or failed.
- In many cases, the operation fails with no reason provided, hangs forever, or even crashes the whole tab.

{% include figure.html
  src="2024-02-gmailctl/gmail-import-failing.gif"
  alt="import failing with no reason provided"
  caption="import failing with no reason provided"
  maxwidth="480px"
%}

### nested labels not really nested

Another surprise is that nested labels (e.g. `news/a`, and `news/b`) do not actually inherit. Therefore, you can't easily filter all mail with labels `news/a` or `news/b`. The only way is to add a parent label `news` to all children mails as well.

## config as code

<a href="https://github.com/antifuchs/gmail-britta">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/ruby/ruby-original.svg" width="24" />
  gmail-britta
</a>
and 
<a href="https://github.com/mbrt/gmailctl">
  <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/go/go-original.svg" width="24" />
  gmailctl
</a>
both attempt to manage Gmail config as code. `gmail-britta` is the pioneer, but unfortunately has been unmaintained, while `gmailctl` is actively maintained.

`gmailctl` uses [jsonnet](https://jsonnet.org/), which is a templating language that compiles json-compatible data. With a templating language, it's possible to define helper functions, reuse common filter actions, dynamically generate labels/filters, and split the configurations into different files.

### sample decent configuration starter set

```jsonnet
// subscriptions.libsonnet

local subscriptions = [
  { sender: "Ruby Weekly" },
  { sender: "LLVM Weekly" },
  // ...
];

local rules = [{
  filter: { from: s.sender },
  actions: {
    markSpam: false,
    markImportant: false,
    category: "updates",
    labels: [ "subscriptions", "subscriptions/" + s.sender ]
  },
} for s in subscriptions];

// enforce the parent label
local labels = [{ name: "subscriptions" }] + [{ name: "subscriptions/" + s.sender } for s in subscriptions];

{
  labels: labels,
  rules: rules,
}
```

```jsonnet
// invoices.libsonnet

local action = {
  markSpam: false,
  markImportant: false,
  category: "updates",
  labels: [
    "invoices"
  ]
};

local rules = {
  {
    filter: {
      or: [
        { from: "Google Payments" },
        { from: "@intl.paypal.com" },
        // ...
      ]
    },
    actions: action,
  },
  {
    filter: {
      and: [
        { from: "Apple" },
        { subject: "receipt" }
      ]
    },
    actions: action,
  },
  {
    filter: {
      and: [
        { from: "Microsoft Azure" },
        { subject: "billing" }
      ]
    },
    actions: action,
  },
}

local labels = [{
  name: "invoices",
  color: {
  background: "#ffad46",
  text: "#ffffff"
  }
}]

{
  labels: labels,
  rules: rules,
}
```

```jsonnet
// config.jsonnet

local lib = import 'gmailctl.libsonnet';

local me = 'my@self.com';

local subscriptions = import 'subscriptions.libsonnet';
local subscriptionLabels = subscriptions["labels"];
local subscriptionRules = subscriptions["rules"];

local invoices = import 'invoices.libsonnet';
local invoiceLabels = invoices["labels"];
local invoiceRules = invoices["rules"];

{
  version: "v1alpha3",
  labels: [
    { name: "Notes" }, // gmail native ones
  ] + subscriptionLabels + invoiceLabels,
  rules: [
    // directly TO me
    {
      filter: lib.directlyTo(me), // a helper to match me only in TO, not in CC/BCC
      actions: {
        markImportant: true
        labels: ["p0", "p0/directed"]
      },
    },
    // CC/BCC me
    {
      filter: {
        or: {
          { cc: me },
          { bcc: me },
        }
      },
      actions: {
        markImportant: true
        labels: ["p1", "p1/involved"]
      },
    },
  ] + subscriptionRules + invoiceRules
}
```

### the migration flow

With this flow, you can migrate your current UI-based Gmail configuration to a declarative one.

1. prepare the configurations
    1. install and setup [gmailctl](https://github.com/mbrt/gmailctl).
    1. download and back up your current config with `gmailctl download > ~/.gmailctl/config.jsonnet`.
    1. read the [docs](https://github.com/mbrt/gmailctl) and compose your own configurations.
    1. verify your change with `gmailctl diff`
1. a fresh start
    1. (optional) if you want to manage labels with gmailctl config as well
        1. (optional, and with caution) remove all labels from all mails in [Gmail setting](https://mail.google.com/mail/u/0/#settings/labels).
        1. use `gmailctl apply` to apply labels. This command also create filters but we can batch-delete them later anyway.
    1. re-apply all filters:
        1. navigate to [Gmail settings/Filter and Blocked Addresses](https://mail.google.com/mail/u/0/#settings/filters).
        1. (with caution) remove all existing filters.
        1. use `gmailctl export > ~/.gmailctl/filters.xml` to export filters into an xml file.
        1. click "Import filters", select the xml file, and then click "Open file".
        1. select filters to import, check "Apply new filters to existing email" and click "Create filters".
            1. NOTE: selecting too many (~10) filters in one batch might cause it to stuck indefinitely. In that case, just refresh and retry.
            1. if the import fails, try to inspect the response with devtool/network. One error I encountered was caused by me disabled the "smart label" feature but included one `category: "updates"` in my rules.
1. back up your `.gmailctl` folder [just like any other dotfiles](https://news.ycombinator.com/item?id=11070797).
