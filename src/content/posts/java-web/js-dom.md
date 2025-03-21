---
title: js-dom
published: 2025-03-18
description: 'js-dom的使用'
image: ''
tags: [javascript, dom]
category: 'java-web'
draft: false 
---
JavaScript DOM（Document Object Model，文档对象模型）是用于操作HTML和XML文档的API。通过DOM，JavaScript可以访问和修改网页的内容、结构和样式。以下是一些基本的DOM操作方法：
### 1. 选择元素
- `document.getElementById(id)`：通过元素ID选择元素。
- `document.getElementsByTagName(tagName)`：通过标签名选择元素集合。
- `document.getElementsByClassName(className)`：通过类名选择元素集合。
- `document.querySelector(selector)`：返回匹配指定选择器的第一个元素。
- `document.querySelectorAll(selector)`：返回匹配指定选择器的所有元素集合。
### 2. 修改元素内容
- `element.innerHTML`：获取或设置元素内部的HTML内容。
- `element.textContent`：获取或设置元素内部的文本内容。
### 3. 修改元素属性
- `element.getAttribute(attributeName)`：获取元素指定属性的值。
- `element.setAttribute(attributeName, value)`：设置元素指定属性的值。
- `element.removeAttribute(attributeName)`：删除元素指定属性。
### 4. 修改元素样式
- `element.style.property`：获取或设置元素的样式属性，例如`element.style.color`。
### 5. 创建和添加元素
- `document.createElement(tagName)`：创建一个新的元素。
- `parentElement.appendChild(childElement)`：将一个元素添加到另一个元素的末尾。
- `parentElement.insertBefore(newElement, referenceElement)`：在参考元素之前插入一个新元素。
### 6. 删除元素
- `parentElement.removeChild(childElement)`：从父元素中删除一个子元素。
### 7. 事件处理
- `element.addEventListener(event, function)`：为元素添加事件监听器。
- `element.removeEventListener(event, function)`：移除元素的事件监听器。
### 示例
以下是一个简单的示例，展示了如何使用DOM来改变一个段落元素的文本内容：
```javascript
// 选择元素
var paragraph = document.getElementById('myParagraph');
// 修改元素内容
paragraph.textContent = '新的段落文本内容';
// 修改元素样式
paragraph.style.color = 'red';
// 添加事件监听器
paragraph.addEventListener('click', function() {
  alert('段落被点击了！');
});
```
在这个示例中，我们首先通过`getElementById`选择了一个ID为`myParagraph`的段落元素，然后使用`textContent`属性修改了它的文本内容，并使用`style.color`属性修改了它的文本颜色。最后，我们为这个段落添加了一个点击事件监听器，当段落被点击时，会弹出一个警告框。
这些只是DOM操作的基础知识，DOM提供了非常丰富的API，可以执行更复杂和强大的操作。随着你对JavaScript和DOM的深入学习，你会掌握更多高级的技术和技巧。
