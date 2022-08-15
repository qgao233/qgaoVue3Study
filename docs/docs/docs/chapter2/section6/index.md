# vue的逻辑处理

## 1 v-if · v-else-if · v-else

元素必须在同级，并且是紧跟上一元素。

这三个在只有当条件为true时，才会渲染元素，也就是dom上才会有这个元素，否则就没有。

```
<button @click="type = 'A'">Toggle</button>
<button @click="type = 'B'">Toggle</button>

<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

### 1.1 `<template>` 上的 v-if

因为 v-if 是一个指令，他必须依附于某个元素。但如果我们想要切换不止一个元素呢？在这种情况下我们可以在一个 `<template>` 元素上使用 v-if，这只是一个不可见的包装器元素，最后渲染的结果并不会包含这个 `<template>` 元素。

```
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

v-else 和 v-else-if 也可以在 `<template>` 上使用。

>思考：可否像这样：
>```
><div v-if="ok">
>  <h1>Title</h1>
>  <p>Paragraph 1</p>
>  <p>Paragraph 2</p>
></div>
>```

## 2 v-show

v-show 仅切换了该元素上名为 `display` 的 CSS 属性，并且会在 DOM 渲染中保留该元素。

v-show 不支持在 `<template>` 元素上使用，也不能和 v-else 搭配使用。

```
<h1 v-show="ok">Hello!</h1>
```

## 3 v-if vs v-show

* v-if 是惰性的：如果在初次渲染时条件值为 false，则不会做任何事。条件区块只有当条件首次变为 true 时才被渲染。
    >v-if 是“真实的”按条件渲染，因为它确保了在切换时，条件区块内的事件监听器和子组件都会被销毁与重建。

* v-show 简单许多，元素无论初始条件如何，始终会被渲染，只有 CSS display 属性会被切换。

总的来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要频繁切换，则使用 v-show 较好；如果在运行时绑定条件很少改变，则 v-if 会更合适。

## 4 v-if 和 v-for

当 v-if 和 v-for 同时存在于一个元素上的时候，v-if 会首先被执行。

>同时使用 v-if 和 v-for 是不推荐的，因为这样二者的优先级不明显。