# `<template>`内的ref

可以称为**模板引用**。

## 1 使用ref

某些情况下，我们仍然需要直接访问底层 DOM 元素。

`ref` 是一个特殊的属性，和 v-for 章节中提到的 key 类似。它允许我们在一个特定的 DOM 元素或子组件实例被挂载结束后，它的直接引用都会被暴露在 `this.$refs` 之上：

```
<script>
export default {
  mounted() {
    this.$refs.input.focus()    //在组件挂载后将焦点设置到一个 input 元素上
  }
}
</script>

<template>
  <input ref="input" />
</template>
```

>注意，只可以在组件挂载后才能访问模板引用。如果你想在模板中的表达式上访问 $refs.input，在初次渲染时会是 null。这是因为在初次渲染前这个元素还不存在

## 2 v-for中使用ref

当在 v-for 中使用模板引用时，相应的引用中包含的值是一个数组：

```
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      { { item } }
    </li>
  </ul>
</template>
```

>注意，ref 数组并不保证与源数组相同的顺序。

## 3 ref作为函数入参

`ref` 还可以绑定为一个函数，会在每次组件更新时都被调用。该函数会收到元素引用作为其第一个参数：

```
<input :ref="(el) => { /* 将 el 赋值给一个数据属性或 ref 变量 */ }">
```

注意我们这里需要使用动态的 `:ref` 绑定才能够传入一个函数。当绑定的元素被卸载时，函数也会被调用一次，此时的 el 参数会是 null。你当然也可以绑定一个组件方法而不是内联函数。

## 4 组件上使用ref

`ref`被用在子组件上时，这种情况下引用中获得的值的是组件实例：

```
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child 是 <Child /> 组件的实例
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

如果一个子组件使用的是**选项式 API** ，被引用的组件实例和该子组件的 `this` 完全一致，这意味着父组件对子组件的每一个属性和方法都有完全的访问权。

>这使得在父组件和子组件之间创建紧密耦合的实现细节变得很容易，当然也因此，应该只在绝对需要时才使用组件引用。

大多数情况下，你应该首先使用标准的 `props` 和 `emit` 接口来实现父子组件交互。

### 4.1 限制对子组件实例的访问

expose 选项可以用于限制对子组件实例的访问：

```
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
}
```

在上面这个例子中，父组件通过模板引用访问到子组件实例后，仅能访问 `publicData` 和 `publicMethod`。