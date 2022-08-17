# 状态变量侦听器

## 1 简单介绍

我们知道计算属性允许我们声明性地计算衍生值。

然而在有些情况下，我们需要在**状态变化时**执行一些“副作用”，例如：

* 更改 DOM，
* 或是根据异步操作的结果去修改另一处的状态。

```
export default {
  data() {
    return {
      question: '',
      answer: 'Questions usually contain a question mark. ;-)'
    }
  },
  watch: {
    // 此处的方法名要与状态变量一致
    // 每当 状态变量question 改变时，这个函数就会执行
    question(newQuestion, oldQuestion) {
      if (newQuestion.indexOf('?') > -1) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.answer = 'Thinking...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Error! Could not reach the API. ' + error
      }
    }
  }
}
<p>
  Ask a yes/no question:
  <input v-model="question" />
</p>
<p>{ { answer } }</p>
```

>watch 选项也支持把键设置成用 . 分隔的路径：
>```
>export default {
>  watch: {
>    // 注意：只能是简单的路径，不支持表达式。
>    'some.nested.key'(newValue) {
>      // ...
>    }
>  }
>}
>```

## 2 深层侦听器

`watch` 默认是**浅层**的：被侦听的属性，仅在被赋新值时，才会触发回调函数——而嵌套属性的变化不会触发。如果想侦听所有嵌套的变更，你需要深层侦听器：

```
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // 注意：在嵌套的变更中，
        // 只要没有替换对象本身，
        // 那么这里的 `newValue` 和 `oldValue` 相同
      },
      deep: true
    }
  }
}
```

>深度侦听需要遍历被侦听对象中的所有嵌套的属性，当用于大型数据结构时，开销很大。因此请只在必要时才使用它，并且要留意性能。

## 3 即时回调的侦听器

`watch` 默认是**懒执行**的：仅当数据源变化时，才会执行回调。但在某些场景中，我们希望在创建侦听器时，立即执行一遍回调。

举例来说，我们想请求一些初始数据，然后在相关状态更改时重新请求数据。

我们可以用一个对象来声明侦听器，这个对象有 handler 方法和 immediate: true 选项，这样便能强制回调函数立即执行：

```
export default {
  // ...
  watch: {
    question: {
      handler(newQuestion) {
        // 在组件实例创建时会立即调用
      },
      // 强制立即执行回调
      immediate: true
    }
  }
  // ...
}
```

## 4 回调的触发时机

默认优先性：用户创建的侦听器回调 `>` Vue 组件更新时机

如果想在侦听器回调中能访问被 Vue 更新之后的DOM，你需要指明 flush: 'post' 选项：

```
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'post'
    }
  }
}
```

## 5 `this.$watch()`

可以使用组件实例的 `$watch()` 方法来命令式地创建一个侦听器：

```
export default {
  created() {
    this.$watch('question', (newQuestion) => {
      // ...
    })
  }
}
```

如果要在特定条件下设置一个侦听器，或者只侦听响应用户交互的内容，这方法很有用。它还允许你提前停止该侦听器。

## 6 停止侦听器

用 watch 选项或者 $watch() 实例方法声明的侦听器，会在宿主组件卸载时自动停止。

在少数情况下，你的确需要在组件卸载之前就停止一个侦听器，这时可以调用 $watch() API 返回的函数：

```
const unwatch = this.$watch('foo', callback)

// ...当该侦听器不再需要时
unwatch()
```