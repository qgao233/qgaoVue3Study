# v-for介绍

## 1 遍历数组

```
data() {
  return {
    parentMessage: 'Parent',
    items: [{ message: 'Foo' }, { message: 'Bar' }]
  }
}

<li v-for="(item, index) in items">     //当前项的位置索引index参数为可选参数
  { { parentMessage } } - { { index } } - { { item.message } }
</li>
```

渲染后：

```
<li>Parent - 0 - Foo</li>
<li>Parent - 1 - Bar</li>
```

### 1.1 js模拟vue的实现效果

```
const parentMessage = 'Parent'
const items = [
  /* ... */
]

items.forEach((item, index) => {
  // 可以访问外层的 `parentMessage`
  // 而 `item` 和 `index` 只在这个作用域可用
  console.log(parentMessage, item.message, index)
})
```

### 1.2 遍历时使用解构

```
<li v-for="{ message } in items">   //从对象里解构出字段
  { { message } }
</li>

<!-- 有 index 索引时 -->
<li v-for="({ message }, index) in items">
  { { message } } { { index } }
</li>
```

### 1.3 `of` 替代 `in`

```
<div v-for="item of items"></div>
```

>JavaScript 的迭代器语法用 of

## 2 遍历对象

```
<script>
export default {
    data() {
        return {
            myObject: {
                title: 'How to do lists in Vue',
                author: 'Jane Doe',
                publishedAt: '2016-04-10'
            }
        }
    }
}
</script>

<template>
    <ul>
        <li v-for="(value, key, index) in myObject">
            { { index } }. { { key } }: { { value } }
        </li>
    </ul>
</template>
```

渲染后：

```
<ul>
    <li>0. title: How to do lists in Vue</li>
    <li>1. author: Jane Doe</li>
    <li>2. publishedAt: 2016-04-10</li>
</ul>
```

## 3 v-for的循环次数

`<span v-for="n in 10">{ { n } }</span>`

注意此处 n 的初值是从 1 开始而非 0，即`n = [1,10]`。

## 4 `<template>` 上的 v-for

与模板上的 v-if 类似，你也可以在 `<template>` 标签上使用 v-for 来渲染一个包含多个元素的块。例如：

```
<ul>
  <template v-for="item in items">
    <li>{ { item.msg } }</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

## 5 v-for 与 v-if

在上一section的最后一个小节讲过，

当它们同时存在于一个节点上时，v-if 比 v-for 的优先级更高。这意味着 v-if 的条件将无法访问到 v-for 作用域内定义的变量别名：

```
<!--
 这会抛出一个错误，因为属性 todo 此时
 没有在该实例上定义
-->
<li v-for="todo in todos" v-if="!todo.isComplete">
  { { todo.name } }
</li>
```

在外新包装一层 `<template>` 再在其上使用 v-for 可以解决这个问题 (这也更加明显易读)：

```
<template v-for="todo in todos">
  <li v-if="!todo.isComplete">
    { { todo.name } }
  </li>
</template>
```

## 6 v-for渲染后又发生更新

>v-for 渲染的元素列表更新的默认模式为：当数据项的顺序改变时，Vue 不会随之移动 DOM 元素的顺序，而是就地更新每个元素内的值，但只适用于列表渲染输出的结果**不依赖子组件状态**或者**临时 DOM 状态** (例如表单输入值) 的情况。

上面仅供了解，如果发生需要移动dom元素的场景，我们使用v-for，通常需要绑定一个`key`跟踪每个节点的标识：

```
<div v-for="item in items" :key="item.id">  //使用的v-bind:key
  <!-- 内容 -->
</div>
```

当你使用 `<template v-for>` 时，`key` 应该被放置在这个 `<template>` 容器上：

```
<template v-for="todo in todos" :key="todo.name">
  <li>{ { todo.name } }</li>
</template>
```

>key 绑定的值期望是一个基础类型的值，例如字符串或 number 类型。不要用对象作为 v-for 的 key。

## 7 组件上使用 v-for

```
<script>
import TodoItem from './TodoItem.vue'
  
export default {
  components: { TodoItem },
  data() {
    return {
      todos: [
        {
          id: 1,
          title: 'Do the dishes'
        },
        {
          id: 2,
          title: 'Take out the trash'
        },
        {
          id: 3,
          title: 'Mow the lawn'
        }
      ],
    }
  }
}

<todo-item
    v-for="(todo, index) in todos"
    :key="todo.id"
    :title="todo.title"
    @remove="todos.splice(index, 1)"
></todo-item>
```

下面是TodoItem.vue

```
<script>
export default {
    props: ['title'],
}
</script>

<template>
  <li>
    { { title } }
  </li>
</template>
```

### 7.1 渲染结果

```
<li>Do the dishes</li>
<li>Take out the trash</li>
<li>Mow the lawn</li>
```

## 8 数组变化时的渲染

### 8.1 变更方法

对原数组进行修改。

```
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

### 8.2 非变更方法

原数组不变，返回一个新数组。

```
filter()
concat()
slice()
```

例：

```
this.items = this.items.filter((item) => item.message.match(/Foo/))
```

### 8.3 利用计算属性

有时，我们希望显示数组经过过滤或排序后的内容，而不实际变更或重置原始数据。

```
data() {
  return {
    sets: [[ 1, 2, 3, 4, 5 ], [6, 7, 8, 9, 10]]
  }
},
methods: {
  even(numbers) {
    return numbers.filter(number => number % 2 === 0)
  }
}
<ul v-for="numbers in sets">
  <li v-for="n in even(numbers)">{ { n } }</li>
</ul>
```

在计算属性中使用 reverse() 和 sort() 的时候务必小心！这两个方法将变更原始数组，计算函数中不应该这么做。请在调用这些方法之前创建一个原数组的副本：

```
//return numbers.reverse()
return [...numbers].reverse()
```