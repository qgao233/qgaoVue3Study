# 何为组件

组件允许我们将 UI 划分为独立的、可重用的部分，并且可以对每个部分进行单独的思考。

Vue 实现了自己的组件模型，使我们可以在每个组件内封装自定义内容与逻辑。

## 1 定义组件

* 单文件组件 (SFC)：将 Vue 组件定义在一个单独的 .vue 文件中。

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
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>
```

* 内联写法：这里的`template`是一个内联的 JavaScript 字符串，Vue 将会在运行时编译它。

```
export default {
  data() {
    return {
      count: 0
    }
  },
  template: `
    <button @click="count++">
      You clicked me {{ count }} times.
    </button>`
}
```

你也可以使用 ID 选择器来指向一个元素 (通常是原生的 `<template>` 元素)，Vue 将会使用其内容作为模板来源。(这句话没懂)

## 2 使用组件

这里仅介绍单文件的使用。

>[两种不同写法的使用方法](https://staging-cn.vuejs.org/examples/)

```
<script>
import ButtonCounter from './ButtonCounter.vue' //在父组件中导入

export default {
  components: {
    ButtonCounter       //为了暴露给父组件的模板使用，需要注册，注册时的名字作为模板中的标签名。
  }
}
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

组件可以重复使用，每使用一次，就创建一个新的实例（参考类与对象的关系）：

```
<h1>Here is a child component!</h1>
<ButtonCounter />
<ButtonCounter />
<ButtonCounter />
```

在单文件组件中，推荐为子组件使用 `PascalCase` （大驼峰写法）的标签名，以此来和原生的 HTML 元素作区分。虽然原生 HTML 标签名是不区分大小写的，但 Vue 单文件组件是可以在编译中区分大小写的。我们也可以使用 /> 来关闭一个标签。

如果你是直接在 DOM 中书写模板 (例如原生 `<template>` 元素的内容)，模板的编译需要遵从浏览器中 HTML 的解析行为。在这种情况下，你应该需要使用 `kebab-case` 形式并显式地关闭这些组件的标签。

```
<!-- 如果是在 DOM 中书写该模板 -->
<button-counter></button-counter>
<button-counter></button-counter>
<button-counter></button-counter>
```

## 3 传递属性 props（父组件与子组件交互）

表示博客文章的组件：

```
<!-- BlogPost.vue -->
<script>
export default {
  props: ['title']  //注册title
}
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

该属性的值 (title) 可以像其他组件属性一样，在模板和组件的 this 上下文中访问。

当一个 prop 在子组件内被注册后，可以在父组件中如此使用：

```
<BlogPost title="My journey with Vue" />
<BlogPost title="Blogging with Vue" />
<BlogPost title="Why Vue is so fun" />
```

在实际应用中不会像上面这样写，而是如下：

```
export default {
  // ...
  data() {
    return {
      posts: [
        { id: 1, title: 'My journey with Vue' },
        { id: 2, title: 'Blogging with Vue' },
        { id: 3, title: 'Why Vue is so fun' }
      ]
    }
  }
}

<BlogPost
  v-for="post in posts"
  :key="post.id"
  :title="post.title"
 />
```

## 4 监听事件（子组件与父组件交互）

要在此处实现 `A11y` 的需求，将博客文章的文字能够放大，而页面的其余部分仍使用默认字号。

>A11y：Accessibility，可访问性的，11代表“A”和“Y”之间的11个字母。它指的是每个人（包括残障人士）对软件的可访问性。

首先在**父组件**中，我们可以添加一个 `postFontSize` 数据属性来控制所有博客文章的字体大小：

```
data() {
  return {
    posts: [
      /* ... */
    ],
    postFontSize: 1
  }
}

<div :style="{ fontSize: postFontSize + 'em' }">
  <BlogPost
    v-for="post in posts"
    :key="post.id"
    :title="post.title"
   />
</div>
```

然后在**子组件**`<BlogPost>`中添加一个按钮：

```
<!-- BlogPost.vue, 省略了 <script> -->
<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <button>Enlarge text</button>
  </div>
</template>
```

我们想要点击这个按钮来告诉父组件它应该放大所有博客文章的文字。

此时，**父组件**可以通过 `v-on` 或 `@` 来选择性地监听子组件上抛的事件，就像监听原生 DOM 事件那样：

```
<BlogPost
  ...
  @enlarge-text="postFontSize += 0.1"
 />
```

**子组件**可以通过调用内置的 $emit 方法，通过传入事件名称来**抛出一个事件**：

```
<!-- BlogPost.vue -->
<script>
export default {
  props: ['title'],
  emits: ['enlarge-text']
}
</script>

<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <button @click="$emit('enlarge-text')">Enlarge text</button>
  </div>
</template>
```

`emits`声明了一个组件可能触发的所有事件，还可以对事件的参数进行验证。同时，这还可以让 Vue 避免将它们作为原生事件监听器隐式地应用于子组件的根元素。

## 5 `<slot>` 占位符

父组件使用引用的组件时：

```
<AlertBox>
  Something bad happened.
</AlertBox>
```

子组件中可以用 `<slot>` 将在父组件中子组件包含的内容引用进来。

```
<template>
  <div class="alert-box">
    <strong>This is an Error for Demo Purposes</strong>
    <slot />
  </div>
</template>

<style scoped>
.alert-box {
  /* ... */
}
</style>
```

<div style="border:1px solid red">
    <p style="color: red;">
        This is an Error for Demo Purposes
    </p>
    <p>Something bad happened.</p>
</div>

## 6 动态组件

有些场景会需要在两个组件间来回切换，比如 Tab 界面：

通过 Vue 的 `<component>` 元素和特殊的 `is` 属性实现：

```
<!-- currentTab 改变时组件也改变 -->
<component :is="currentTab"></component>
```

被传给 `:is` 的值可以是以下几种：

* 被注册的组件名
* 导入的组件对象

你也可以使用 `is` 属性来创建一般的 HTML 元素。

当使用 `<component :is="...">` 来在多个组件间作切换时，被切换掉的组件会被卸载。我们可以通过 `<KeepAlive>` 组件强制被切换掉的组件仍然保持“存活”的状态。

## 7 直接在DOM中写模板

即不是在vue中写？

如果你想在 DOM 中直接书写 Vue 模板，Vue 则必须从 DOM 中获取模板字符串。由于浏览器的原生 HTML 解析行为限制，有一些需要注意的事项。

>请注意下面讨论只适用于直接在 DOM 中编写模板的情况。如果你使用来自以下来源的字符串模板，就不需要顾虑这些限制了：
>
>* 单文件组件
>* 内联模板字符串 (例如 `template: '...'`)
>* `<script type="text/x-template">`

### 7.1 大小写区分

HTML 标签和属性名称是不分大小写的，所以浏览器会把任何大写的字符解释为小写。这意味着当你使用 DOM 内的模板时，无论是 PascalCase 形式的组件名称、camelCase 形式的 prop 名称还是 v-on 的事件名称，都需要转换为相应等价的 kebab-case (短横线连字符) 形式：

```
// JavaScript 中的 camelCase
const BlogPost = {
  props: ['postTitle'],
  emits: ['updatePost'],
  template: `
    <h3>{{ postTitle }}</h3>
  `
}
<!-- HTML 中的 kebab-case -->
<blog-post post-title="hello!" @update-post="onUpdatePost"></blog-post>
```

### 7.2 闭合标签

Vue 的模板解析器支持任意标签使用 /> 作为标签关闭的标志：

```
<MyComponent />
```

然而在 DOM 模板中，我们必须显式地写出关闭标签：

```
<my-component></my-component>
```

### 7.3 元素位置限制

某些元素仅在放置于特定元素中时才会显示，例如 `<li>`，`<tr>` 和 `<option>`。

这将导致在使用带有此类限制元素的组件时出现问题。例如：

```
<table>
  <blog-post-row></blog-post-row>
</table>
```

自定义的组件 `<blog-post-row>` 将作为无效的内容**被忽略**，因而在最终呈现的输出中造成错误。我们可以使用特殊的 `is` 属性作为一种解决方案：

```
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```

>当使用在原生 HTML 元素上时，is 的值必须加上前缀 vue: 才可以被解析为一个 Vue 组件。这一点是必要的，为了避免和原生的自定义内置元素相混淆。