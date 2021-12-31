---
layout: single
title: "Javascript: Prototype and Object-oriented"
excerpt: 由若干 key-value pairs 組成的結構，在 ruby 裏稱 `hash`，在 javascript 中叫做 `object`
date: 2014-09-12 23:25
---

# 1 What is and what is not an object

由若干 key-value pairs 組成的結構，在 Ruby 裏稱 `Hash`，在 Python 裏稱爲 `Map`，在 Javascript 中叫做 `Object`。

但 ruby 中還有 ruby 的 object，javascript 中再沒有另一個「對象」，javascript 只有 object，它同時扮演了在其他語言中的不同概念：`hash`，`object`，`class`，甚至 `property`，`function` 和 `method`。如果處於普遍意義上的 Object-oriented 語境中，則可說它既是 class，也是 instance，甚至也是 method。

有人據此認爲「javascript 中一切都是 object」，這種說法容易產生誤解。

Javascript 中，每個變量都屬於一個類型，可以用 `typeof a` 來「算出」變量的類型。

  *<small>說「算出」，是因爲 typeof 是一個 operator。</small>*

`typeof` 的運算結果會是以下幾種之一：`undefined`，`null`，`number`，`string`，`boolean`，`symbol`（ES2015 新增），以及 `function` 和 `object`。

這似乎在提示我們，一個字符串，或是一個數字，並非是一個 object。但這就無法解釋爲什麼可以執行 `'1'.toString()` 這樣的語句，因爲 `oneVar.oneMethod()` 意味着調用在 `oneVar` 上名爲 `oneMethod` 的 `method`，而一個字符串只是一個單純的值，並非是 key-value 的結構，也就不具有包含或是調用 method 的能力。

簡單來講，`Null`，`Undefined`，`String`，`Number`，`Boolean`，以及 `Symbol`，是 6 個 `primitive type`，如果一個值的類型是這樣的 primitive type，它原本是沒有通常意義上的 method 的。

而此六種 primitive type 中，除了 Null 和 Undefined 之外，都有與之同名的 wrapper，例如 `String` 就是一個全局的 object，它也就是創建字符串變量時被調用的 constructor。這就解釋了爲什麼一個字符串作爲 primitive value，可以調用 toString() method，而一個值爲 `undefined` 或 `null` 的變量卻不可以。

  *<small>例如 `'1'.constructor` 的結果是 'function String() { [native code] }'，有關 constructor 的詳細內容會在後面提到。</small>*

  *<small>雖然 primitive 變量的值是 immutable 的（不能被修改，只能賦予一個新的值），但是上述同名 wrapper 是可以被修改的，因此可以通過 `String.prototype.toString = function() {return 'compromised'}` 來修改所有字符串的 toString method。</small>*

接下來講到的所謂 object，通常是指 typeof 結果爲 'object' 的非 primitive 變量。

值得注意的是，function 並非是一種 primitive type。後面會提到，函數只是稍有特殊的 object。

  *<small>另一個細節是，NaN，Infinity 與 Null 和 Undefined 不同，前兩項並非 primitive 類型，它們僅是類型爲 'number' 的變量的一種值。</small>*

Javascript 中，object 與 string 等 non-primitive type 的另一個顯著區別是，如果一個變量的值是一個 object，那麼它實際僅存儲了這個 object 的 reference（地址／引用），而如果變量的值是一個字符串或一個數字，則存儲了這個值本身。通俗的解釋是，如果兩個變量的值是同一個 object，那麼修改一個變量會同時修改另一個變量。這是個常見的問題，在此不再多做解釋。

# 2 First constructor

```
var student = { name: 'a common name' }
```

這行代碼定義了一個普通的 object。它有一個名爲 name 的 property，property attribute 是一個字符串變量。

可以用 student.name 或 student['name'] 來訪問或修改 name 的值，也可以執行 `student.toString()`。這意味着存在兩個 method 作爲 getter 和 setter，也存在着一個名爲 toString 的 method。那麼，一個自然的問題就是：這些 method 在哪裏？

我們知道，上面一條語句相當於執行了：

```
var student = new Object()
student.name = 'a name'
```

這裏的 `new` 也是一個 operator。而 `Object` 是一個 constructor。就像在 C++ 或 ruby 中，class 是構造 object 的模板；而在 javascript 中，人們用一個叫做 prototype 的 object 作爲 constructor 來構建另一個 object。

*<small>因此人們說 javascript 是 prototype-based。而 C++ 則是 object-based。</small>*

*<small>這裏一個更重要的區別是，class 定義後無法修改，而 prototype 則相反。這種運行時可以改變自身結構的特性，叫做 meta-programming，通常說這種語言是 dynamic 的。</small>*

更具體地講，一個 constructor 是一個 function（本質上也就是一個 object），它有一個叫做 prototype 的 property，這個 property 的 attribute 又是一個 object，它用來作爲生成新 object 的模板。

試着構造一個 constructor：

```
var Student = function() {}
Student.prototype.name = 'a common name'
```

用它來創建新 object

```
var bob = new Student()
bob.name // 'a common name'
bob.name = 'Bob'
bob.name // 'Bob'
```

當調用`new Student()` 時，大略發生了以下事情：

1. 新建一個 object，不妨稱它爲 ref

2. ref 是以 Student.prototype 爲模板構造的，它「複製」了 Student.prototype 的所有 property 作爲自己的 inherited property，這一過程就是所謂的 inherit。

    *<small>這裏的「複製」需要注意，之前提到過，如果是 object（包括 function）的話，那麼只複製了 reference，這意味着如果對其進行修改的話，也會對其 constructor 中相同 object 或 function 產生影響，反之亦然，如果修改了 constructor.prototype 的 method，那麼 instance 也會受到影響。</small>*

3. 在 ref 的 context 下調用此函數。

    *<small>所謂 context，即 execution context，大略指運行時所處的變量環境，以及 this 的值。在內部實現時，這裏必然有一個複雜的處理 context 的過程，但在此可以簡單的理解成：首先令 ref.constructor 的值爲 Student() 函數，然後再行調用。而我們瞭解，調用 object method 時，函數中的 `this` 即是這一 object 本身。</small>*

    *<small>關於 execution context 的內容請參見 [ES 5.1: 10.3](http://www.ecma-international.org/ecma-262/5.1/#sec-10.3)</small>*

    *<small>上述代碼中我們沒有傳遞參數到 Student() 中，但如果有的話，（例如 `new Student(arg)`），在調用 Student() 時也會包含相同的參數。</small>*

4. 爲了方便的維護 object 的繼承關係，Javascript 引擎會在 ref 上維護一個名爲 [[prototype]] 的 internal property。

    *<small>這裏會令人感到疑惑，後面會詳細解釋。</small>*

5. 最後將 ref 作爲 `new Student()` 的返回值。

    *<small>如果 Student() 函數內的返回值是一個 object 的話，那麼這個 object 將代替 ref 成爲 new 語句的返回值。但是通常不會這麼做。</small>*

過程有些複雜，而且真正在內部實現起來的過程只會更複雜。但我們現在只需記住大概過程，其中的重點是，新 object「繼承」了 constructor 的 prototype。

BTW，上面提到在 new 時 Student 函數會被執行，而執行時 `this` 是被新建的 object，那麼就有了另一種構造 constructor 的方式：

```
var Student = function(name) {
  this.name = name
}
```

請自行模擬在執行 `var bob = new Student('Bob')` 後會發生什麼。

# 3 Instanceof and Prototype chain

上述內容中，有一個容易令人感到迷惑之處：我們遇到了兩個 prototype。

第一個是 constructor 的 prototype（即 Student.prototype），它用來作爲模板構造 instance；

另一個是每個 object 都有的名爲 [[prototype]] 的 internal property，它的值要麼是 null，要麼就是自身繼承自的 prototype，也即構造自身所用的 constructor.prototype。

有時人們所說的「變量 bob 的 prototype」就是指這個 [[prototype]]。暫且把它記作 bob.[[prototype]]，當然在代碼裏這樣是行不通的。因爲作爲 internal property，按照標準它對於用戶是不可見的。

*<small>然而是在一些 implementation 上（比如幾乎所有瀏覽器），提供了 `__proto__` 來訪問和操作 [[prototype]]，但很多人都不推薦使用它。</small>*

雖然無法直接訪問 bob.[[prototype]]，但是可以使用 `Object.getPrototypeOf(bob)` 來查看 bob 繼承自何物。

  *<small>ES 6 中新增了 Object.setPrototypeOf，雖然依舊是很多人所不推薦使用的。</small>*

總結一下，其實你只需要記住一句話：「constructor 的 prototype 是 instance 的 prototype」。

  *<small>即 Student.prototype === Object.getPrototypeOf(bob) === true</small>*

  *<small>當然，constructor 也可能有自己的 constructor 和自己的 [[prototype]]。</small>*

  *<small>對我來講，之所以這裏曾經令我費解，是因爲我不理解 prototype 這個詞的本義。我瞭解什麼是 OOP 中的 constructor，instance，和 inherite，但在決定深入瞭解 Javascript 之前，我完全沒有 prototype 這個概念。一些時候，面對技術問題，詞典和同義詞典也能起到妙用。</small>*

解決了令人疑惑的概念，我們回到之前的地方。

現在我們得到了一個名爲 bob 的 object。

這時執行 `bob instanceof Student` 可以得到結果是 `true`。

instanceof 又是一個 operator。顧名思義，`bob instanceof Student` 的意思就是 `is bob an instance of Student?`。

通俗的說法是，constructor 與 instance 之間是一種「互逆」的關係。bob 的 constructor 是 Student，bob 也就是基於 Student 構造出的 instance。

但這個說法並不完全準確，因爲 bob 只能有一個 constructor，但是 Student 可能構造出無數個 instance。更何況，還要考慮 constructor 的 constructor 的問題。也就是，Z 繼承自 X，X 繼承自 Y，……，B 繼承自 A。這時 Z 既 `instanceof A`，也 `instanceof Y`。這時互逆的關係就可能不再成立。

顯然，這種繼承的關係是一個鏈（chain）式結構，我們之前瞭解到，這種關係是通過 [[prototype]] 來維護的。事實上，這種關係就叫做 `prototype chain`。

接下來試着實現一個簡單的兩重繼承關係：Person -> Student -> bob

首先先實現兩個 constructor

```
var Person = function() {
  this.name = 'a common name'
}

var Student = function(school) {
  this.school = school
}
```

現在這兩個 constructor 還沒有聯繫起來，我希望 Student 能繼承自 Person，於是：

```
Student.prototype = new Person()
```

這是一行關鍵而神奇的代碼。首先使用 Person constructor 新建了一個 person instance。而 Student 作爲一個 constructor，它的 prototype，也就是用來 construct 其他 instance 的模板，現在就是這個 person instance。

這意味着，用 Student 構造新的 student instance 時，首先會繼承所有的 Person 的屬性，而在執行 Student() 函數的時候，又給自身添加了 `school` 屬性。

```
var bob = new Student('NEYC')
bob.name   // 'a common name'
bob.school // 'NEYC'
```

接下來驗證繼承鏈最重要的性質：即如果我修改了 Person constructor，那麼 Student constructor 也會受到影響，反之則不然：

```
Person.prototype.showName = function() {
  console.log(this.name)
}

Student.prototype.showSchool = function() {
  console.log(this.school)
}

bob.showName() // 'a common name'

alice = new Person()
alice.showName()   // 'a common name'
alice.showSchool() // TypeError: undefined is not a function
```

Prototype chain 由 Javascript 引擎負責維護。當訪問 bob 上的 method 或者 property 時，會先在 bob 自身查找，如果沒有的話，會沿着 [[prototype]] 上溯。

*<small>這提示我們：可以在鏈條的下端進行 override。</small>*

接下來驗證一下 instanceof 的結果：

```
bob instanceof Person  // true
bob instanceof Student // true
```

這說明，在進行 `object instanceof constructor` 運算時，同樣會沿着 object 的 prototype chain 上溯，依次檢查鏈中之物是否是 constructor.prototype。

提問：此時 `Object.getPrototypeOf(bob)`（即 bob.[[prototype]]）的值爲何？

# 4 Truely Object-oriented

前面模擬了簡單的繼承關係。但顯然距離 object-oriented 還有相當差距。

接下來嘗試模擬一個更完善的，勉強可以被稱爲 object-oriented 的 subclass -> superclass 結構。

先不忙寫代碼，讓我們先來確認一下具體有哪些需求：

*<small>以下是 OOP 語境中的概念，毋與 Javascript 中的概念混淆（如 constructor）。</small>*

1. 能夠定義 class 作爲模板來生成 instance，其中包含 property 與 method。
2. class 中應有 constructor，在建立 instance 時執行。
3. Inheritance，subclass 能夠繼承 superclass，並且不會影響到上游。
4. `super` 關鍵字以訪問 superclass 的內容。
5. Namespace，在不同的 namespace 中定義的 class 不會互相污染。

以下 feature 實現起來過於繁雜，先行忽略：

1. Information hiding，能夠控制一個 method 或 property 的訪問權限，例如將其設爲 private 的。
2. Polymorphism，根據不同的參數數目與類型，可以定義多個同名的 constructor，property，或 method。
3. Abstract class，或稱 virtual class，即無法被 instantiate 的 class。

先把上一節中的代碼做些改進：

```
function Person() {}

Person.prototype.setName = function(name) {
  this.name = name
  console.log('my name is now ' + name)
}

function Student() {
  Person.call(this) // KEYPOINT 1
}

Student.prototype = Object.create(Person.prototype) // KEYPOINT 2
Student.prototype.constructor = Student // KEYPOINT 3

var bob = new Student()

bob.setName('Bob')
```

與上一節中的代碼相比，有三處不同。

## 4.1 keypoint 1

在 subclass（Student）的 constructor 內部，執行了 `Person.call(this)`。

之前提到，全局對象 String，是所有字符串變量的 wrapper；同樣，全局對象 Function，是所有 function 的 constructor，在 Function.prototype 中定義了若干所有函數都具備的 property 與 method，比如 `call`。

顧名思義，`call` 用來調用其它函數。但更重要的是，用戶可以指定這一函數運行時所處的 context。簡單的理解，你可以指定是由哪個 object 調用的這一函數；最直觀的作用是，你可以指定運行時的 `this`。方法 `call` 所接受的第一個參數，就是指定的 `this`。

  *<small>`call` 的具體語法請參見 [MDN: Function.call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)</small>*

依前所述，此處的 `Student()` 是通過 `var bob = new Student()` 來被調用的。那麼此時的 `this` 就是剛剛新建的對象 `ref`，即隨後的 `bob`。於是，`Person.call(this)`，就是以 `ref` 爲 `this` 來調用 Person()。

這裏有一個問題。在前一節中，我們並沒有顯示的 call Person，但是似乎也沒有遇到什麼問題，那麼此舉意義何在？這要和下一點放在一起來說，請先暫且讀下去。

## 4.2 keypoint 1 & 2

上一節中 `Student.prototype = new Person()`，等號右側現在變成了 `Object.create(Person.prototype)`，這種改變的原因是什麼。

  *<small>Object.create 的具體內容後面還會講到。現在只需瞭解 `Object.create` 接受一個 prototype 作爲參數，並據此生成一個新的 object。</small>*

  *<small>我相信你現在可以理解我說「一個 prototype」中 prototype 的含義。</small>*

首先，到目前位置我們的 constructor Person 還是不接受參數的，而如果接受參數的話，在執行 `Student.prototype = new Person()` 時，我們還不知道應該傳遞什麼參數進去。

其次，`new Person()` 會使得 `Person()` 被執行一次，而在前面我們已經進行了一次 `Person.call(this)`。這樣的話 `Person()` 會被執行兩次。而且從邏輯上講，也應該是在調用 subclass 建立對象時再新建 superclass 比較合理。

因此，這裏的關鍵是，`Object.create` 僅僅按照 prototype 新建了 object，而不會執行 constructor 函數本身。

梳理一下可以發現，keypoint 1 和 2 其實是同一處改進。

按照之前對 `new` 具體執行過程的探索，可以瞭解到，在上一節中的 `new Person()` 隱含了兩個步驟：

1. 建立一個新 object
2. 以此新 object 的名義調用 Person()

而我們現在把兩者拆開分別執行：

1. Object.create(Person.prototype)
2. Person.call(this)

  *<small>一个值得考慮的問題：將兩者分開更多的優勢。</small>*

  *<small>可以參見這個問題 [StackOverflow: using-object-create-instead-of-new](https://stackoverflow.com/questions/2709612/using-object-create-instead-of-new)。</small>*

  *<small>這個問題中答案的內容和本文目前的過程並不同步，建議讀者先繼續閱讀本文，最後一同進行發散閱讀。</small>*

## 4.3 keypoint 3

觀察 `Student.prototype.constructor = Student` 一句。

我們之前見到過 `constructor` 這一 property，但從未操作過它的值。

回憶一下 `.constructor` 的相關內容：

```
var Person = function() { console.log('Person constructor') }
Person.constructor  // function Function() { [native code] }

var bob = new Person()
bob.constructor // 即 Person 本身。

var Student = function() { console.log('Student constructor') }
Student.prototype = Object.create(Person.prototype)

bob = new Student()
bob.constructor // 依然是 Person 本身。
```

可以看到，如果不手動維護 `.constructor` 的話，那麼只有最頂層的 superclass 擁有內建的 `.constructor`，而其餘 subclass 的 `.constructor` 均爲沿 prototype chain 繼承而得。因此，我們需要手動 override 這一值。

## 4.4 Uncomplete inheritance

目前的繼承並不完整。

這是指，subclass 只繼承了 superclass 的 prototype 中的 method 或 property，而這並不包括 superclass 自身的 method 或 property。也就是 `Person.prototype.foo` 與 `Person.bar` 間的區別，我們的 subclass Student 通過 `Person.prototype` 繼承，自然不包括 `bar`。

那麼就是手動爲 subclass 添加一個同名的 `bar`。

```
Student.bar = Person.bar
```

可以使用 for 循環遍歷 object 的所有 property 與 method
```
for (var key in Person){
  Student[key] = Person[key]
}
```

Well… NO!

因爲這時 for in 循環的結果中也包含了 `Person.prototype` 中定義的 property，還有繼承而來的 `.constructor` 等等，我們只想遍歷真正在 Person 「名下」的 property，此時需要`hasOwnProperty` 這一 method。

```
for ( var key in Person ) {
  if ( Person.hasOwnProperty( key ) ) {
    Student[key] = Person[key]  
  }
}
```

*<small>`hasOwnProperty()` 這一 method 是定義在 `Object.prototype` 中的。功能即字面含義。</small>*

*<small>讀到這裏沒發現什麼不自然之處麼？爲什麼在 for 循環中，`hasOwnProperty` 本身沒有被遍歷到？還有熟悉的 `toString()` 呢？這個後面再說。</small>*

## 4.5 Super

在 Java 和 Ruby 等諸多語言中，關鍵字 `super` 用來表示當前 class 的 superclass，可以通過它來訪問 superclass 的 property 與 method。

在解決了之前繼承不完全的問題之後，要實現類似 `super` 的效果就很簡單了。

```
Student.superClass = Student.prototype
```

  *<small>`super` 一詞是 Javascript 中的 reversed word，在 ES 5.1 中它沒有實際作用，在 ES 6 中被用於錯誤處理方面。因而在代碼中不妨以 superClass 代替。</small>*

## 4.6 Namespace

前面的東西都相對複雜，下面我們來實現一個相對簡單的 feature：namespace。

一言以蔽之：不要忘記 constructor 本質上還是一個 function，將其作爲一個 object 名下的 method，這一 object 自然就滿足 namespace 的要求。

## 4.7 More　

關於在 JS 中實現 OOP 的內容，還可以參見以下篇章（複雜程度依次上升，最後一例則特殊：它不包含詳細的解說，但其實相當優雅而易讀）：

1. [MDN: Introduction to Object-Oriented JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript)
2. [phrogz.net: OOPinJS](http://phrogz.net/js/classes/OOPinJS.html)
3. [ejohn.org: Simple Javascript Inheritance](http://ejohn.org/blog/simple-javascript-inheritance/)
4. [CoffeeScript: Class, Inherite and Super](http://coffeescript.org/#classes) 

# 5 A brand new object

之前多次用到了 `Object`，在結束之前簡單地說明一下。

```
var anEmptyObject = {}
anEmptyObject.toString() // '[Object Object]'
```

可見，一個「空」的 object 並非真的一無所有，它已經包含若干 method。一個自然的想法是，它亦是繼承自他物。

那麼來看它繼承自何物：

```
anEmptyObject.constructor            // function Object() { [native code] }
Object.getPrototypeOf(anEmptyObject) // Object {}
```

`Object` 是 Javascript 中的一個內建全局對象，就像 `String` 是字符串類型變量的 wrapper 一樣，`Object` 是 object 類型變量的 wrapper。但 `Object` 還有些其他的用途。

之前提到過 `Object.create()` 用來根據一個 prototype 新建 object。執行 `var a = {}` 或 `var a = new Object()` 時，相當於執行了 `var a = Object.create(Object.prototype)`。其中 `Object.prototype` 就包含了 `hasOwnPrototype()`，`toString()` 等 method。但這時，如果傳入一個 `null` 作爲 prototype，那麼我們就得到了一個真正的空 object。 

*<small>`Object.prototype` 包含的具體內容請參見：[ES 5.1: sec-15.2.3.1](http://www.ecma-international.org/ecma-262/5.1/#sec-15.2.3.1)，及 [MDN: Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)</small>*

我們從這個真正的空 object 來開始實驗：

```
var anTruelyEmptyObject = Object.create(null)
```

現在我們來用更底層的方式來爲這個 object 添加一個 property

```
Object.defineProperty(anTruelyEmptyObject, 'name', {
  value: 'Sailor Moon',
  writable: true,
  enumerable: true,
  configurable: true
});
```

`Object.defineProperty()` method 功能同名，此處在 anTruelyEmptyObject 上定義了一個名爲 name 的 property。重點是第三個參數，其中 `value` 好理解，其餘三個則會令後學者困惑。

事實上，對於一個 object 中的一個 property 來講，首先它有自己的 propertyName，其次還有 4 個 property attribute：`value`，`writable`，`enumerable`，及 `configurable`。`writable` 決定其值是否可被修改；`configurable` 決定其 property attribute 是否可被更改，以及 property 本身是否可被刪除；重點是 `enumerable`，它決定該 property 是否在 `for (key in anTruelyEmptyObject)` 時出現，這就是之前遇到的問題的答案。

如果每次爲 object 添加 property 都要這麼做的話就太麻煩了，於是 Javascript 提供了簡化的方式。`anTruelyEmptyObject.name = 'Sailer Moon'` 一句，其背後即隱示地調用了 `Object.defineProperty` 方法，其中三個 boolean 值均爲默認值 true。

`anTruelyEmptyObject.name = {name: 'Sailer Moon'}` 是更加簡化的版本。但是要注意的是，使用 {} 來新建的 object，其 [[prototype]] 會被賦爲 `Object.prototype`。有時這並非是件好事，比如之前在新建 constructor 時。

於是我們可以自己實現一個 helper，以簡單地完成「由 prototype 建立 object，並設置 property」這一過程：

```
var createObjectFromPrototype = function(prototype, object) {
  var newObject = Object.create(prototype)

  for (var key in object) {
    if (object.hasOwnProperty(key)) {
      newObject[key] = object[key]
    }
  }

  return newObject
}

var Person = {
  toString: function() {
    console.log('overrided toString method')
    return(this.name)
  }
};

var bob = createObjectFromPrototype(Person, {
  name: 'Bob'
})

bob.name // 'Bob'
```
這樣我們就簡單地通過 constructor 而 override 了 toString() 方法。
