---
title: vue入门
published: 2025-03-20
description: 'vue的基本使用'
image: ''
tags: [vue]
category: 'java-web'
draft: false 
---
**Vue.js 基本用法**
Vue.js 是一个用于构建用户界面的渐进式JavaScript框架。它的核心库只关注视图层，不仅易于学习，还容易与其它库或已有项目整合。
**1. 创建Vue实例**
首先，你需要通过`<script>`标签引入Vue.js。然后，创建一个Vue实例，指定一个挂载点（通常是CSS选择器或HTML元素），并在数据对象中定义属性。
```html
<div id="app">{{ message }}</div>

<script type="module">
  import { createApp, ref } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'

  createApp({
    setup() {
      const message = ref('Hello Vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```
**2. 指令**
Vue.js 使用指令来扩展HTML标签的功能。指令带有`v-`前缀，表示它们是Vue特有的属性。
**常用指令：**
* `v-bind`：动态地绑定一个或多个属性，或一个组件prop到表达式。可以简写为`:属性名`，例如`v-bind:src=""`可以简写为`:src=""`。
* `v-if`/`v-else-if`/`v-else`：条件渲染块。
* `v-show`：和`v-if`类似，但切换的是元素的CSS属性`display`。
* `v-for`：基于一个数组渲染一个列表。
* `v-on`：绑定事件监听器到元素上。
* `v-model`：在表单输入和应用状态之间创建双向绑定。
**v-for 指令**

`v-for` 指令用于基于一个数组渲染一个列表。它的基本语法是`v-for="item in items"`，其中`items`是源数据数组，而`item`是正在遍历的数组元素的别名。
```html
<ul>
  <li v-for="item in items">
    {{ item.text }}
  </li>
</ul>
```
**最佳实践**
* **组件化**：将界面分解为独立可复用的组件。
* **单向数据流**：避免直接修改props，使用事件或回调来更新父组件的状态。
* **计算属性**：使用计算属性来处理复杂的逻辑，而不是在模板中直接写表达式。
* **监听器**：使用`v-on`或`@`符号来监听事件，并触发方法。
* **模板语法**：避免在模板中写复杂的JavaScript代码，保持模板简洁。
* **性能优化**：使用`v-if`和`v-show`明智地，避免不必要的渲染。
* **代码组织**：使用模块系统（如ES6模块或CommonJS）来组织代码。
* **测试**：为组件编写单元测试，确保代码质量和可维护性。
**示例**：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>简化版Vue示例</title>
    <style>
        .hidden { display: none; }
    </style>
</head>
<body>
    <div id="app">
        <!-- v-model示例：双向绑定输入框 -->
        <input type="text" v-model="message" placeholder="输入消息" />
        <p>{{ message }}</p>

        <!-- v-bind示例：动态绑定图片地址 -->
        <img :src="imageUrl" alt="动态图片" />

        <!-- v-if/v-else-if/v-else示例：条件渲染 -->
        <div v-if="number > 10">数字大于10</div>
        <div v-else-if="number < 10">数字小于10</div>
        <div v-else>数字等于10</div>

        <!-- v-show示例：切换显示状态 -->
        <button v-on:click="toggleVisibility">切换显示/隐藏</button>
        <div v-show="isVisible">这是一个可切换显示的元素</div>

        <!-- v-for示例：列表渲染 -->
        <ul>
            <li v-for="item in items" :key="item.id">{{ item.text }}</li>
        </ul>
    </div>

    <script type="module">
        import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'
        createApp({
            data() {
                return {
                    message: '', // v-model绑定的消息
                    imageUrl: 'https://example.com/image.png', // v-bind绑定的图片地址
                    number: 5, // v-if条件渲染使用的数字
                    isVisible: true, // v-show控制显示状态的变量
                    items: [ // v-for渲染的列表数据
                        { id: 1, text: '列表项1' },
                        { id: 2, text: '列表项2' },
                        { id: 3, text: '列表项3' }
                    ]
                }
            },
            methods: {
                toggleVisibility() { // v-on绑定的事件处理函数
                    this.isVisible = !this.isVisible;
                }
            }
        }).mount('#app')
    </script>
</body>
</html>

```
在这个示例中，我们创建了一个简单的待办事项列表，其中包含了添加和删除功能。我们使用了`v-model`来实现双向绑定，`v-for`来渲染列表，以及`v-on`来监听事件。同时，我们遵循了组件化和单向数据流的最佳实践。
