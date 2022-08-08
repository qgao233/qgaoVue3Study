# 创建一个 Vue 应用

## 1 应用实例

`createApp`返回一个**应用实例**。

```
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})
```

### 1.1 传入参数

传入的参数称之为这个应用实例的**根组件**，

```
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```

而在这个根组件内部import到的其他组件称之为子组件，如下：

```
App (root component)
├─ TodoList
│  └─ TodoItem
│     ├─ TodoDeleteButton
│     └─ TodoEditButton
└─ TodoFooter
   ├─ TodoClearButton
   └─ TodoStatistics
```

## 2 根组件实例

应用实例通过调用 `.mount()` 方法后才被渲染出来，返回的称为**根组件实例**。

### 2.1 传入参数

传入参数称为**容器**，可以是：

* 一个实际的 DOM 元素 **(标签)**
* 或一个 CSS 选择器字符串 **(id/class)**

```
<div id="app"></div>
app.mount('#app')
```

`createApp`传入的根组件的内容将会被渲染在容器元素里面。而容器元素自己将不会被视为应用的一部分。

当在未采用构建流程的情况下使用 Vue 时，我们可以在挂载容器中直接书写根组件模板：

```
<div id="app">
  <button @click="count++">{{ count }}</button>
</div>
import { createApp } from 'vue'

const app = createApp({
  data() {
    return {
      count: 0
    }
  }
})

app.mount('#app')
```

当根组件没有设置 template 选项时，Vue 将自动使用容器的 innerHTML 作为模板。

>.mount() 方法应该始终在整个应用配置和资源注册(见3 应用配置)完成后被调用。

## 3 应用配置

应用实例会暴露一个 .config 对象允许我们配置一些应用级的选项，例如定义一个应用级的错误处理器，它将捕获所有由子组件上抛而未被处理的错误：

```
app.config.errorHandler = (err) => {
  /* 处理错误 */
}
```

应用实例还提供了一些方法来注册应用范围内可用的资源，例如注册一个组件：

```
app.component('TodoDeleteButton', TodoDeleteButton)
```

这使得 TodoDeleteButton 在应用的任何地方都是可用的。

## 4 多个应用实例

应用实例并不只限于一个。createApp API 允许你在同一个页面中创建多个共存的 Vue 应用，而且每个应用都拥有自己的用于配置和全局资源的作用域。

```
const app1 = createApp({
  /* ... */
})
app1.mount('#container-1')

const app2 = createApp({
  /* ... */
})
app2.mount('#container-2')
```

如果你正在使用 Vue 来增强服务端渲染 HTML，并且只想要 Vue 去控制一个大型页面中特殊的一小部分，应避免将一个单独的 Vue 应用实例挂载到整个页面上，而是应该创建多个小的应用实例，将它们分别挂载到所需的元素上去。