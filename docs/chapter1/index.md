# vue3入门

## 示例

`test.vue`

```
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```

## 1 俩特点

* 声明式渲染：Vue 基于标准 HTML 拓展了一套模板语法，使得我们可以声明式地描述最终输出的 HTML 和 JavaScript 状态之间的关系。
    >就是说：把各部分要用到的功能描述了一遍，由底层框架去调用程序员声明的功能实现来运行程序。
* 响应性：Vue 会自动跟踪 JavaScript 状态变化并在改变发生时响应式地更新 DOM。
    >说白了，一些定义的变量产生变化，就自动刷新局部页面。

## 2 组件

和react中的概念类似，这里写好这里可以用，也可以被其他地方导入，在其他地方使用。

## 3 单文件组件

使用一种类似 HTML 格式的文件来书写 Vue 组件，它被称为单文件组件 (也被称为 *.vue 文件，英文 Single-File Components，缩写为 SFC)。

说白了，Vue 的单文件组件(一个文件中) = 逻辑 (JavaScript) + 模板 (HTML) + 和样式 (CSS) 

## 4 组件书写风格

### 4.1 组合式 API

`setup`是一个标识，告诉 Vue 需要在编译时进行一些处理，让我们可以更简洁地使用组合式 API。比如，`<script setup>` 中的导入和顶层变量/函数都能够在`<template>`中直接使用。

```
<script setup>
import { ref, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 用来修改状态、触发更新的函数
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log(`The initial count is ${count.value}.`)
})
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### 4.2 选项式 API

类似面向对象语言，选项式 API 以“组件实例”的概念为中心 (即例子中的 this指向当前的组件实例)，基于组合式 API 的基础上实现。

```
<script>
export default {
  // data() 返回的属性将会成为响应式的状态
  // 并且暴露在 `this` 上
  data() {
    return {
      count: 0
    }
  },

  // methods 是一些用来更改状态与触发更新的函数
  // 它们可以在模板中作为事件监听器绑定
  methods: {
    increment() {
      this.count++
    }
  },

  // 生命周期钩子会在组件生命周期的各个不同阶段被调用
  // 例如这个函数就会在组件挂载完成后被调用
  mounted() {
    console.log(`The initial count is ${this.count}.`)
  }
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### 4.3 如何选用

* 当你不需要使用构建工具，或者打算主要在低复杂度的场景中使用 Vue，例如渐进增强的应用场景，推荐采用选项式 API。

* 当你打算用 Vue 构建完整的单页应用，推荐采用组合式 API + [单文件组件](../chapter2/section13/index.md)。

## 5 quick start

注意：在本地用vite构建vue项目时，使用`npm init vue@latest`会在当前目录创建你的项目，因此选好目录再使用该命令最好。