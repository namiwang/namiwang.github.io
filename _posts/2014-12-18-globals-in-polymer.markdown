---
layout: single
title: "Globals in Polymer"
excerpt: Polymer 中 element（component）主宰一切。一個自然的需求是，在 element 間進行通訊與數據共享。
date: 2014-12-18 16:56:04 +0800
---

## 0 Introduction

Polymer 中 element（component）主宰一切。一個自然的需求是，在 element 間進行通訊與數據共享。

## 1 Communication

### 1.1 Published property

Polymer 會自動根據 element 的 `attributes="valueA"`生成相應的 getter，setter，以及 `change watcher`（即`@valueAChanged(oldValue, newValue)`），這就是所謂的`published property`。這實質上就是 element 的 public API。

通過調用其它 element 的 API 就是最基本的通訊方式。即，在 elementA 中：

```
@.$.elementB.valueB = 'foo'
```

而在 elementB 中使用 watcher：

```
Polymer
  valueBChanged: (oldValue, newValue) ->
    console.log "valueB changed from #{oldValue} to #{newValue}"  
```

### 1.2 core-signals

上述通過 public attribute 進行通訊的方式十分繁瑣。Polymer 中提供了`core-signals`用以進行通訊：

```
<link rel="import" href="bower_components/core-signals/core-signals.html">
<polymer-element name="my-app">
  <template>
    <core-signals on-core-signal-event-foo="{{ handle_event_foo }}"></core-signals>
    <child-element></child-element>
  </template>
  <script>
    Polymer
      handle_event_foo: (e, message, sender) ->
        console.log "event foo got fired with message: #{message}"
  </script>
</polymer>

<polymer-element name="child-element">
  <template>
    <paper-button on-click="{{ fire_foo }}">fire_foo</paper-button>
  </template>
  <script>
  switch_to_world_map: ->
    @fire 'core-signal',
      name: 'event-foo'
      data: 'one message'
  </script>
</polymer-element>
```

*代碼並不嚴謹，比如沒有 import element，而且實際上也不能直接在`<script>`中寫 coffee，僅作示意之用。*

有關`core-signals`還有幾點額外說明：

1. 一個`core-signals`可以接收多個 signal，即

        <core-signals
          on-core-signal-event-a="{{ handle_event_a }}"
          on-core-signal-event-b="{{ handle_event_b}}">

2. signal 是發往全局的，多個 element 也可以接收同樣的 signal，它們都會被觸發。
3. 可用`asyncFire`替代`fire`來進行異步觸發。
4. 如果很多 element 需要引入`<core-signals>`，可以建立一個`<base-element>`然後 extend 它。

### 1.3 Others
理論上也可以使用其它 Javascript 通用的方式進行通訊，例如 sharedWorker 等。此處不展開。

## 2. Global sharing
### 2.1 Published property
依然是最基本的方式，在使用 element 時通過 attributes 共享 object，即：

```
<polymer-element name="parent-element" attributes="valueFoo">
  <template>
    <child-element valueFoo="valueFoo"></child-element>
  </template>
  <script>
    Polymer
      ready: ->
        valueFoo = { foo: 'bar' }
  </script>
</polymer-element>
```

其中 child-element 有 attributes="valueFoo"。

### 2.2 A custom global element

文檔中[此處](https://www.polymer-project.org/docs/polymer/polymer.html#global)介紹了一種在全局共享數據的方式。具體來講，可以定義一個名爲 app-globals 的 element，通過 closure，使得所有的<app-globals>的 instance 都共享同樣的數據：

```
<polymer-element name="my-globals" attributes="userName">
  <script>
  (function() {
    var userName = 'Farmer John'

    Polymer({
       ready: function() {
         this.userName = userName;
       }
    })
  })()
  </script>
</polymer-element>
```

*如果使用 coffee 的話，默認即有此 closure wrapper。*

此時在其它任何 elemet 中，只要引入<my-globals>即可獲取其中的數據。
```
<polymer-element name="my-app">
  <template>
    <my-globals id="globals"></my-globals>
  </template>
  <script>
    Polymer
      ready: ->
        console.log @.$.globals.userName
  </script>
</polymer-element>
```

但是，這裏還有兩個問題：
1. 目前只能支持一個名爲`userName`的變量。更多的變量就要寫成諸如`attributes="valueA valueB valueC"`的形式。
2. 基於 javascript 的特性，如果此處共享一個字符串變量，那麼在一個 element 中對其進行修改是無法傳遞到全局的，即這隻是一個全局共享的只讀變量。

這兩個問題的答案是一樣的：使用一個 object 來包裝共享的數據：

```
<polymer-element name="my-globals" attributes="values">
  <script>
    values = {}
    Polymer
      ready: ->
        @values = values
  </script>
</polymer-element>
```

如此，我們便構造了一個全局共享的，可讀可寫的 object。

此時，我們已經可以通過`<span>{{ globals.values.userName }}</span>`來在頁面中直接獲取全局共享的值。但若要修改其中的值，還需要使用代碼（雖然本刊名爲《代碼指南》，但我們依然不喜歡寫無用的重複的代碼）。我們希望可以隨時直接使用`<my-globals userName="Farmer John"></my-globals>`來修改全局變量。顯然，爲達此目的，我們需要能夠獲取 element 的 attributes，此時需要的是 Polymer 在 element 上維護的`this.attributes`值。我們作出如下修改：

```
<polymer-element name="my-globals" attributes="values">
  <script>
    values = {}
    Polymer
      ready: ->
        @values = values
        for attr in @attributes
          @values[attr.nodeName] = attr.value
  </script>
</polymer-element>
```

此時，我們既可以在頁面中使用`<my-globals userName="Farmer John"></my-globals>`，也可以在代碼中使用`@.$.globals.values.userName = 'Farmer John'`來修改全局變量。

更多的討論：

1. 其中的`@attributes`，即是 Polymer 在 element 上維護的`this.attributes`。對於`<my-element vertical layout foo="bar"></my-element>`，其`this.attributes`大致形如：

    ```
    {
        0: { nodeName: 'vertical', value: '' },
        1: { nodeName: 'layout', value: '' },
        2: { nodeName: 'foo', value: 'bar' }
    }
    ```

2. 同`<core-signals>`一樣，可以定義`<base-element>`，在其中引入`<my-globals>`，其它 elements 對 base 進行 extend。

3. 實踐中，可能需要在 values 有修改的時候進行操作。比如，實時地將其值存入 localStorage。

    此時如果要使用一個`change watcher`（即`@valuesChanged`）來監視`values`的變化，那麼一定要注意：如果 watch 的對象是一個 object，那麼對 object 進行修改`@values.foo = 'bar'`，是**不會觸發 watcher** 的。只有修改 values 本身（`@values = 'new value as a string'`，纔會觸發 watcher。

    而 Polymer 中的`observe`雖然可以 watch「某 object 某 attr」的變化，但是這需要我們事先知道該 attr 的名稱，通用性不足。

    因此實踐中，可以在`<my-globals>`上手動定義 getter 與 setter 作爲操作數據的接口：`<my-globals attributes="set_values get_values"></my-globals>`，然後在`set_values` method 中進行其它操作，舉例如下：

    ```
    set_values: (hash) ->
      new_values = @.$.localStorage.value
      for key of hash
        new_values[key] = hash[key]
      @.$.localStorage.value = new_values
      @.$.localStorage.save()  
    ```

相關閱讀：

- [https://www.polymer-project.org/docs/polymer/polymer.html#global](https://www.polymer-project.org/docs/polymer/polymer.html#global)

### 2.3 core-meta

Polymer 提供了`core-meta`來進行簡單的數據存儲。其在功能上類似於我們在上一節中實現的全局共享的變量，但是更加輕量：

要添加或修改一個全局變量，可以在任意位置引用：
```
<core-meta id="user" userName="Farmer John"></core-meta>
```

要讀取一個值，也要先有一個`<core-meta>`，可以是直接添加在 html 中的，也可以在需要讀取數據時動態添加：
```
meta = document.createElement('core-meta');
# 通過 id 來讀取一個值：
console.log meta.byId 'user' # Farmer John
# 列出所有定義過的<core-meta>
console.log meta.metaArray
# 列出所有定義過的 meta 值
console.log meta.metaData
```

一個`<core-meta>`中還可以包含更多的值：
```
<core-meta id="user" userName="Farmer John">
  <property name="lastName" value="John"></property>
</core-meta>
```

*目前，這種方式定義的 property，要得到其值似乎有些繁瑣，需要手動遍歷`meta.metaArray`所返回的 DOM*

相關閱讀：

- [https://www.polymer-project.org/docs/elements/core-elements.html#core-meta](https://www.polymer-project.org/docs/elements/core-elements.html#core-meta)
- [https://github.com/Polymer/core-meta](https://github.com/Polymer/core-meta)
- [http://stackoverflow.com/questions/25774548/appropriate-use-of-metadata-in-polymer-application-i-e-core-meta-element](http://stackoverflow.com/questions/25774548/appropriate-use-of-metadata-in-polymer-application-i-e-core-meta-element)

### 2.4 core-shared-lib

項目變得複雜之後，需要共享的往往不止有數據，還有代碼本身。

`core-shared-lib`用於引用支持 JSONP 的 javascript 代碼：

```
<core-shared-lib on-core-shared-lib-load="{{load}}" url="https://apis.google.com/js/plusone.js?onload=%%callback%%">
```

在實踐中不推薦使用此方法，應儘量使用`<script src="lib.js"></script>`，因爲它會破壞 Polymer（實際上是 webcomponents，以後應該是由瀏覽器）所維護的 imports。

更多內容參見文檔：

- [https://www.polymer-project.org/docs/elements/core-elements.html#core-shared-lib](https://www.polymer-project.org/docs/elements/core-elements.html#core-shared-lib)
- [https://github.com/Polymer/core-shared-lib/blob/master/core-shared-lib.html](https://github.com/Polymer/core-shared-lib/blob/master/core-shared-lib.html)

相關閱讀：

- [http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about)
