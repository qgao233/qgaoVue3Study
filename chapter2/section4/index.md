# 可重用的计算

## 1 computed

```
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  }
}

<span>{ { author.books.length > 0 ? 'Yes' : 'No' } }</span>
```

如果在模板中需要不止一次这样的计算，便显得代码臃肿。

推荐使用**计算属性**`computed`来描述依赖响应式状态的复杂逻辑

```
export default {
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    // 一个计算属性的 getter
    publishedBooksMessage() {
      // `this` 指向当前组件实例
      return this.author.books.length > 0 ? 'Yes' : 'No'
    }
  }
}

<span>{ { publishedBooksMessage } }</span>
```

Vue 会检测到 this.publishedBooksMessage 依赖于 this.author.books，

所以当 this.author.books 改变时，任何依赖于 this.publishedBooksMessage 的绑定都将同时更新。

## 2 computed vs methods

```
// 组件中
methods: {
  calculateBooksMessage() {
    return this.author.books.length > 0 ? 'Yes' : 'No'
  }
}

<p>{ { calculateBooksMessage() } }</p>
```

两者的不同之处在于计算属性值会基于其响应式依赖被缓存。

一个计算属性仅会在其响应式依赖更新时才重新计算。这意味着只要 author.books 不改变，无论多少次访问 publishedBooksMessage 都会立即返回先前的计算结果，而不用重复执行 getter 函数。

**错误使用案例**

```
computed: {
  now() {
    return Date.now()   //Date.now() 并不是一个响应式依赖，因此计算属性now永远不会更新
  }
}
```

## 3 更改计算属性

在外部手动修改计算属性，而不是通过计算函数得出结果，必须得重写`getter`、`setter`方法：

```
export default {
  data() {
    return {
      firstName: 'John',
      lastName: 'Doe'
    }
  },
  computed: {
    fullName: {
      // getter
      get() {
        return this.firstName + ' ' + this.lastName
      },
      // setter
      set(newValue) {
        // 注意：我们这里使用的是解构赋值语法
        [this.firstName, this.lastName] = newValue.split(' ')
      }
    }
  }
}

this.fullName = 'John Doe'      //修改成功，因为setter调用时，实际上还是改的状态变量，从而导致依赖它两个的计算变量得到更新。
```

>总的来说，虽然可以更改，但其实没有必要。

## 4 注意

不要在计算函数中产生异步请求或者更改 DOM等副作用，有专门讲的[监听器]()：根据其他响应式状态的变更来主动创建副作用。