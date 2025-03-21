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
    <title>Vue指令示例</title>
    <style>
      .avatar {
        width: 50px;
        height: 50px;
        border-radius: 50%;
      }
    </style>
</head>
<body>
    <div id="app">
        <h1>用户列表</h1>
        <ul>
            <!-- 使用v-for指令遍历users数组，并使用v-bind指令绑定数据 -->
            <li v-for="(user, index) in users" :key="index">
                <!-- 使用v-bind指令绑定src属性 -->
                <img :src="user.avatar" alt="用户头像" class="avatar">
                <!-- 插值表达式显示用户名字 -->
                <span>{{ user.name }}</span>
            </li>
        </ul>
    </div>

    <script type="module">
      import { createApp } from 'https://unpkg.com/vue@3/dist/vue.esm-browser.js'
      createApp({
        data() {
          return {
            users: [
              { name: '张三', avatar: 'https://example.com/avatar1.jpg' },
              { name: '李四', avatar: 'https://example.com/avatar2.jpg' },
              { name: '王五', avatar: 'https://example.com/avatar3.jpg' }
            ]
          }
        }
      }).mount('#app')
    </script>
</body>
</html>

```
在这个示例中，我们创建了一个简单的待办事项列表，其中包含了添加和删除功能。我们使用了`v-model`来实现双向绑定，`v-for`来渲染列表，以及`v-on`来监听事件。同时，我们遵循了组件化和单向数据流的最佳实践。
