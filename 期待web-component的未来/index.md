# 期待Web Component的未来


Web Component 本身不是一个规范，而是一组技术的应用，Web Component 现阶段主要分为三部分：`Custom element`、`Shadow DOM`、`Template`。通过它们的搭配使用，可以让我们在不借助第三方框架（react，vue）的情况下，构建独立的，可重用的自定义组件。

## Custom element(自定义元素)

Custom element 是一组 javasrcipt API，它允许我们自定义一些定制元素及行为。

```html
<body>
  <!-- 组建使用 -->
  <box-div color="orange" text="hello"></box-div>
</body>

<!-- 组建定义 -->
<script>
  class boxDiv extends HTMLElement {
    constructor() {
      super();
      this.style.color = this.getAttribute('color');
      this.append(this.getAttribute('text'));
    }
  }
  window.customElements.define('box-div', boxDiv);
</script>
```

![demo](01.png 'demo')

js 提供了`customElements.define` api，用来注册自定义元素，customElements.define()接收三个参数：

- 符合命名标准的参数名称（不能是单个单词，且必须要有短横线）
- 用于定义行为的类
- 可选参数：一个包含  `extends`属性的配置对象，指定所创建元素继承于哪个内置的 HTML 元素。

Custom element 有两种定义元素的方式，没有传递第三个参数表示独立创建，不继承内置元素。在使用时可以直接写成 HTML 标签的形式来使用，例如`<word-count>` 或者通过 js 创建`document.createElement(“word-count”)` 。

若在定义元素时指定了所扩展的元素，表示继承创建。在使用时则需要先写出基本的元素标签，并通过`is` 指定所扩展的元素。

```jsx
customElements.define('word-count', WordCount, { extends: 'p' });

// 使用方式
<p is="word-count"></p>;
// 或
document.createElement('p', { is: 'word-count' });
```

它还可以指定一些回调函数，它们将在元素不同的生命时期被调用

- `connectedCallback`：当 custom element 首次被插入文档 DOM 时，被调用。
- `disconnectedCallback`：当 custom element 从文档 DOM 中删除时，被调用。
- `adoptedCallback`：当 custom element 被移动到新的文档时，被调用。
- `attributeChangedCallback`: 当 custom element 增加、删除、修改自身属性时，被调用。

```jsx
class boxDiv extends HTMLElement {
	static get observedAttributes() {
    return ['class', 'style'];
  }
  constructor() {
    super();
    this.style.color = this.getAttribute('color');
    this.append(this.getAttribute('text'));
  }
	public connectedCallback() {
	  console.log('元素插入DOM中.');
	}
	public disconnectedCallback(){
		console.log('元素被删除.');
	}
	public adoptedCallback(){
		console.log('元素被移动.');
	}
	public attributeChangedCallback(name, oldValue, newValue){
		console.log('元素属性变更.' name，oldValue， newValue);
	}
}
```

> 注：想在某个元素属性变化后，触发`attributeChangedCallback()`回调函数，需要先必须监听这个属性。这可以通过定义`observedAttributes()` get 函数来实现，`observedAttributes()`
> 函数体内包含一个 return 语句，返回一个数组，包含了需要监听的属性名称。监听后。每当元素的属性变化时，`attributeChangedCallback()`回调函数会执行。我们可以查看属性的名称、旧值与新值。

通过这些生命周期函数，我们可以动态的变更我们的 web 组件。

## Shadow DOM（影子 DOM）

Shadow DOM 也有一组 javascript API，他不是一个新事物，它允许我们在常规 DOM 元素上附加一个独立隐藏的特殊类型 DOM 元素。这部分的代码与外部代码相互隔离。内部的任何代码都无法影响外部。

通过 Shadow DOM 和 Custom element 搭配的方式，可以保证自定义元素的独立。使其不会影响到它外部的元素。

![shadow 分析](02.png 'shadow 分析')

### **\*\*\*\***\*\***\*\*\*\***基本用法**\*\*\*\***\*\***\*\*\*\***

可以使用 Element.attachShadow() 方法来将一个 shadow root 附加到任何一个元素上。它接受一个配置对象作为参数，该对象有一个 mode 属性，值可以是 open 或者 closed：

```jsx
let shadow = elementRef.attachShadow({ mode: 'open' });
let shadow = elementRef.attachShadow({ mode: 'closed' });
```

`open` 表示可以通过页面内的 javascript 方法来获取 Shadow DOM。

```jsx
let myShadowDom = myCustomElem.shadowRoot;
```

如果你将一个 Shadow root 附加到一个 Custom element 上，并且将 mode 设置为 closed，那么就不可以从外部获取 Shadow DOM 了。`myCustomElem.shadowRoot` 将会返回 null。

### 实现**\*\*\*\***\*\***\*\*\*\***\*\***\*\*\*\***\*\***\*\*\*\***一个简单的 tooltip**\*\*\*\***\*\***\*\*\*\***\*\***\*\*\*\***\*\***\*\*\*\***

```jsx
<body>
    <popup-info text="一个点击可直接暴富的朴素按钮">button</popup-info>
  </body>
  <script>
    class PopUpInfo extends HTMLElement {
      constructor() {
        super();

        const shadow = this.attachShadow({ mode: 'open' });

        const style = document.createElement('style');
        style.textContent = `
      .wrapper {
        position: relative;
        top: 100px;
        left: 10px;
      }
      .info {
        font-size: 14px;
        width: 200px;
        display: inline-block;
        border: 1px solid black;
        padding: 10px;
        background: white;
        border-radius: 10px;
        opacity: 0;
        transition: 0.6s all;
        position: absolute;
        bottom: 20px;
        left: 10px;
        z-index: 3;
      }
      .title:hover + .info, .title:focus + .info {
        opacity: 1;
      }
    `;
        shadow.appendChild(style);

        const wrapper = document.createElement('span');
        wrapper.setAttribute('class', 'wrapper');
        shadow.appendChild(wrapper);

        // title信息
        const titleDom = document.createElement('span');
        titleDom.setAttribute('class', 'title');
        const title = this.innerHTML;
        titleDom.append(title);
        wrapper.appendChild(titleDom);

        // tooltip信息
        const infoDom = document.createElement('span');
        infoDom.setAttribute('class', 'info');
        const text = this.getAttribute('text');
        infoDom.textContent = text;
        wrapper.appendChild(infoDom);
      }
    }

    customElements.define('popup-info', PopUpInfo);
  </script>
```

定义一个自定义 DOM，然后创建一个 shadowDOM，给 shadowDOM 新增子标签`style` `titleSpan` `infoSpan` 。

最后将 shadowDOM 附加到自定义 DOM 上。一个简单的 tooltip 就实现了。

![tooltip组件效果 动图](03.gif '效果 动图')

目前看起来还有一些问题，比如我写 style 的方式是用模版字符串写的。比如我用的 jsAPI 生成的 dom 元素。可读性较差。再比如目前的提示信息只接收字符串，不支持 DOM 类型。这些问题可以使用 template 和 slot 来解决。

## HTML template（HTML 模板）

HTML 提供了<template>和<slot>标签，<template>标签和其子内容不会在 DOM 中呈现。但仍然可以使用 JS 去引用它。借助这个特性。可以让我们创建一个用来灵活填充 web 组件的模版。

```jsx
<template id="my-paragraph">
  <p>My paragraph</p>
</template>
```

这段代码不会在页面中呈现，但却可以通过`document.querySelector()` 等 API 获取

```jsx
let template = document.querySelector('#my-paragraph');
```

<slot>标签则允许我们在模版中定义占位符，在使用该模版时，该占位符可以填充所需的任何 HTML 标记片段。

通过`<template>` 标签和插槽`<slot>`，我们可以将想要的元素定制成一个模版。随取随用。

### **\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***在 web 组件 中使用模版和插槽**\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\***\*\*\*\***\*\***\*\***\*\***

```jsx
<popup-info>
  <a href="http://www.baidu.com" slot="info"
        >一个点击可直接暴富的朴素按钮</a
      >
  <span slot="title">button</span>
</popup-info>
<template id="tooltip-template">
  <style>
    .wrapper {
      position: relative;
      top: 100px;
      left: 10px;
    }
    .info {
      font-size: 14px;
      width: 200px;
      display: inline-block;
      border: 1px solid black;
      padding: 10px;
      background: white;
      border-radius: 10px;
      opacity: 0;
      transition: 0.6s all;
      position: absolute;
      bottom: 20px;
      left: 10px;
      z-index: 3;
    }
    .title:hover + .info,
    .title:focus + .info {
      opacity: 1;
    }
  </style>
  <span class="wrapper">
    <span class="title"><slot name="title">title</slot></span>
    <span class="info"><slot name="info">info</slot></span>
  </span>
</template>

<script>
    class PopUpInfo extends HTMLElement {
      constructor() {
        super();

        const template = document.querySelector('#tooltip-template');
        let templateContent = template.content;

        const shadow = this.attachShadow({ mode: 'open' });
        shadow.append(templateContent.cloneNode(true));
      }
    }

    customElements.define('popup-info', PopUpInfo);
  </script>
```

因为 style 较长，也可以采用 link 标签方式在外部引入

## Web Component 实践

### 基础组件库

市面上已经有很多基于 Web Components 实现的跨框架 UI 组件库。 它可以同时在任意框架或无框架中使用。

### Svelte

scelte 是目前比较火的前端框架，他有一个特点就是可以自定义组件转成通用的 web 组件（web component）。在多团队协同完成的大项目中，各个团队可能使用不同的框架版本，甚至不同的框架，这让不同项目之间的组件复用变得困难。这种情况下 Svelte 就变成了沟通跨越框架鸿沟的桥梁，使用 Svelte 开发的无框架依赖的 Web Components，可以在各个框架间复用。

### 微前端

微前端有几个基本概念： 技术栈无关、应用间隔离、独立开发。目前 Web Components 都符合。在一些微前端方案里就采用了 Web Component 的模式。

…

## Web Component 的缺点

- 再日常开发中，难免会遇到需要调整组件内部样式的时候，但由于 shadowDOM 的隔离机制，会导致我们很难去修改内部的样式。
- 在一些复杂的组件中，数据通信和事件传递存在一定使用成本。
- …

## 结语

Web Component 虽早在 11 年就已推出，且一直发展至今。他的潜力有目共睹，但目前还有很长的路要走。
或许在以后，我们将不再依赖第三方框架，直接使用原生技术来开发页面也说不定。

end.

