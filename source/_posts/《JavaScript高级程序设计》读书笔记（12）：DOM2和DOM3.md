---
title: 《JavaScript高级程序设计》读书笔记（12）：DOM2和DOM3
date: 2019-02-14 16:26:28
categories:
  - 《JavaScript高级程序设计》读书笔记
tags:
  - JavaScript
  - DOM
---

DOM1 主要定义的是 HTML 和 XML 文档的底层结构，DOM2 和 DOM3 级则在此基础上引入更多交互功能，并支持更高级的 XML 特性，并分为不同模块子集：核心、视图、事件、样式、遍历和范围、HTML。

---

## 1. DOM 变化

#### XML 命名空间

针对 XML 命名空间， 不同 XML 文档元素可以混合在一起，而不会发生命名冲突，不同命名空间下的元素拥有各自的子元素，以及各自元素的所有特性。解决了文档中存在两个或多个命名空间时，创建、查询元素（或特性）时不同命名空间归属问题。

#### 其他变化

主要是为了加强 API  的可靠性及完整性。
1. `DocumentType`（文档类型声明） 新增了3个冷门属性：`publicId`、`systemId`、`internalSubset`。
2. 作为 DOM2级核心 -- Document 类型变化
    - 方法 `importNode()`，与命名空间无关，可用来从一个文档中取得一个节点，然后将其导入到另一个文档中，可以跨文档。用法类似于 `cloneNode()`。
     - 属性 `defaultView`(老版IE是 `parentWindow()`) ，该属性保存的指针，指向拥有给定文档的窗口 window。
    - `document.implementation.createDocumentType()` 和
  `document.implementation.createDocument()`，使用后者创建新文档时，可以在参数中传入使用前者所创建的文档类型，因为既有文档的文档类型不能改变，因此 `createDocumentType` 也只有在创建新文档时才有用，但所创建的新文档只有文档元素，没有其余元素。
    - `document.implementation.createHTMLDocument()` 可以创建完整的 HTML 文档实例，唯一参数为文档标题。
3. Node 类型主要有用的变化：
    `isSameNode()` 和 `isEqualNode()`，前者判断是否相同（两个节点引用的是同一个对象），后者判断是否相等（各方面类型、子节点、属性均相等）。
4. 框架的变化：
    框架`HTMLFrameElement` 和 内嵌框架`HTMLIFrameElement` 有了新属性 `contentDocument`，是一个指向表示框架内容的文档对象指针。

---

## 2. 样式

#### 访问元素样式

1. 使用 style 特性定义的内联样式：
    - `myDiv.style.backgroundColor = 'red'` 直接读写(属性名是驼峰)；
    - 使用 style 对象上的API（属性名非驼峰）：
        ```
        var prop = myDiv.style[2]; // 或 myDiv.style.item(2) 返回第3条css属性规则 'background-color'
        var value = myDiv.style.getPropertyValue(prop); // 'red' 返回属性值的字符串表示
        myDiv.style.setProperty('background-color', '#fff') // 设置style特性中的属性
        myDiv.style.removeProperty('background-color') // 删除属性
        myDiv.style.cssText  = 'width: 25px; height: 100px'; // 对 style 对象中的 cssText属性赋值，可以同时应用多项变化
        ```
2. 计算后的样式：
    使用 `document.defaultView.getComputedStyle()` 方法获得可能受到了其他样式表层叠而来的最后的 style 对象， 和元素上的 `style` 属性一样继承自 `CSSStyleDeclaration`对象：
    ```
     document.defaultView.getComputedStyle(myDiv, null) // 第二个参数是伪元素字符串，如： ':after'。
    ```
   计算样式是只读的，修改计算后样式对象中的CSS属性会报错，怎么办呢？可以继续阅读接下来介绍的操作样式表中的 CSS 规则。

#### 操作样式表

`CSSStyleSheet` 类型表示的是样式表，包括 link 和 style 元素中定义的样式表，两种方式可以获取方式样式表对象：一种是`document.styleSheets` 获取应用于当前文档的所有样式表集合，一种是在 link 或 style 元素上的 `sheet`（IE支持 `styleSheet`）属性获取当前样式表。
其继承自 `StyleSheet` 类型，后者作为基础接口用来定义非 CSS 样式表。
`CSSStyleSheet` 对象除了 `disabled` 属性外，其他属性都是只读的，和 style 对象以及 DOM 绝大部分集合一样同样拥有 item() 和 length。

1. CSS 规则：
    `CSSStyleRule` 类型继承自 `CSSRule` 基类对象，表示样式表中的一条 CSS 样式规则信息（一对儿 { } 花括号中的全部内容），除了 CSS 样式规则外，还有 `@import`、`@font-face`、`@page` 和 `@charset` 等，只是那些通常没必要通过脚本访问。
    `CSSStyleRule` CSS样式规则对象的获取方式：
    ```
    var sheet  = document.styleSheets[0]; //  第一个样式表
    var rules = sheet.cssRules || sheet.rules; // 后者为了兼容IE，取得规则列表
    var rule = rules[0]; // 取得第一条规则，也就是第一对儿 { } 花括号定义的规则
    ```
    `CSSStyleRule` 对象中主要属性为：
    | 属性 | 描述 |
    |-|-|
    | `cssText` | 返回整条规则对应文本，和 style.cssText 不同的是前者还包含了选择符文本和花括号，并且只读 |
    | `selectorText` | 返回当前规则的选择符文本 |
    | `style` | 一个 `CSSStyleDeclaration` 对象 |
    关于 `style` 属性，和直接在元素上通过属性 `style` 获取的一样，可以通过它读取和修改规则中的样式信息：
    ```
    // ...
    console.log(rule.style.backgroundColor); // 'red'
    rule.style.backgroundColor = 'yellow'; // 修改当前规则的背景色
    ``` 
    但需要注意，这种通过某个 CSS 样式规则的 style 属性修改样式信息，会影响到页面中所有适用于该规则的所有元素。

2. 添加和删除规则：
    - 向样式表中添加规则，使用 `insertRule()` (<=IE8 可使用 `addRule()`)；
    - 向样式表中删除规则，使用 `deleteRule()` (<=IE8 可使用 `removeRule()`)；
    但是对已有样式表进行添加和删除规则在实际开发中很少见，而且影响 CSS 层叠效果，慎用！

#### 元素大小

1. 偏移量：
    | 属性 | 描述 |
    |-|-|
    | `offsetHeight` | 元素在垂直方向上占用的像素大小，包括水平滚动条高度和边框 |
    | `offsetWidth` | 元素在水平方向上占用的像素大小，包括垂直滚动条高度和边框 |
    | `offsetLeft` | 元素的左外边框至包含元素的左内边框之间的像素距离 |
    | `offsetTop` | 元素的上外边框至包含元素的上内边框之间的像素距离 |
    `offsetLeft` 和 `offsetTop` 属性与包含元素`offsetParent` 有关，`offsetParent` 的值不一定就是父元素 `parentNode`。当 `offsetParent` 为 `body` 时情况比较特殊：
    > 在IE8/9/10及Chrome中，offsetLeft = (body的margin-left)+(body的border-width)+(body的padding-left)+(当前元素的margin-left)。
    > 在FireFox中，offsetLeft = (body的margin-left)+(body的padding-left)+(当前元素的margin-left)。

    ![](https://upload-images.jianshu.io/upload_images/7038854-f5c78608b1e662bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 客户区大小：

    `clientWidth` 和 `cilentHeight` 属性可以获取元素内容及其内边距所占据的空间大小，即 content + padding（不包括滚动条）。
    ![](https://upload-images.jianshu.io/upload_images/7038854-769b9f5fd30d86ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    `document.documentElement.clientWitdth` 可以获取浏览器视口（html元素）大小。

3. 滚动大小：
    | 属性 | 描述 |
    |-|-|
    | `scrollHeight` | 没有滚动条时元素内容总高度，近似于 `clientHeight` |
    | `scrollWidth` | 没有滚动条时元素内容总宽度，近似于 `clientWidth` |
    | `scrollLeft` | 被滚动后左侧隐藏的像素数，可写 |
    | `scrollHeight` | 被滚动后右侧隐藏的像素数，可写 |

    ![](https://upload-images.jianshu.io/upload_images/7038854-dbf92f7ba0106e2b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    确定文档总宽高时，需要取得 `scrollWidth` 或 `clientWidth` 和 `scrollHeight` 或 `scrollHeight` 的最大值，才能获得跨浏览器的精确结果，比如：
    ```
    var html = document.documentElement;
    var docHeight = Math.max(html.scrollHeight, html.clientHeight);
    ```

4. 确定元素大小：

    使用 `element.getBoundingClientRect()` 方法返回元素的大小及其相对于视口的位置。返回的 `DOMRect` 对象是一组用于描述边框属性的集合。
    ![](https://upload-images.jianshu.io/upload_images/7038854-a0bf3021ededde7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    ```
    {
        bottom: 278
        height: 240  // 值等于元素的 offsetHeight，即 content + padding + border
        left: 62
        right: 402
        top: 38
        width: 340  //  值等于元素的 offsetWidth
        x: 62
        y: 38
    }
    ```
    除了 `width` 和 `height` 外的属性都是相对于视口（即当前屏幕文档窗口）的左上角位置而言的，而不是绝对的，因此 `top` 和 `left` 值会随着滚动发生变化。
    如果需要获得相对于整个网页左上角定位的属性值，那么只要给top、left属性值加上当前的滚动位置（通过window.scrollX和window.scrollY），这样就可以获取与当前的滚动位置无关的值，为了跨浏览器兼容，使用 `window.pageXOffset` 和 `window.pageYOffset` 代替 `window.scrollX` 和 `window.scrollY`。