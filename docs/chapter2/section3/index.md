# 如何使页面动态变化

响应式状态：当一个变量通过计算发生值的变化时称为状态的变化，并且页面引用该值的位置会及时得到响应进行改变。

## 1 声明状态变量

选用选项式 API 时，会用 `data` 选项来声明组件的响应式状态。

```
data() {},
```

此选项的值应为返回一个对象的函数。

```
data() {
    return 函数【该函数返回一个对象】
},
```

Vue 将在创建新组件实例的时候调用此函数，并将函数返回的对象用响应式系统进行包装。此对象的所有顶层属性都会被代理到组件实例 (即方法和生命周期钩子中的 `this`) 上。如下：

```
export default {
  data() {
    return {        //直接返回对象也行
      count: 1
    }
  },

  // `mounted` 是生命周期钩子，之后我们会讲到
  mounted() {
    // `this` 指向当前组件实例
    console.log(this.count) // => 1

    // 数据属性也可以被更改
    this.count = 2
  }
}
```

这些实例上的属性仅在实例首次创建时被添加，若所需的值还未准备好，在必要时也可以使用 `null`、`undefined` 或者其他一些值占位。

不在 `data` 上定义的组件实例后添加的新属性将无法触发响应式更新。

>状态变量中不能使用 **$** 或 **_**
> 
>因为Vue 在组件实例上暴露的内置 API 使用 **$** 作为前缀。它同时也为内部属性保留 **_** 前缀。

### 1.1 注意

状态变量基于js代理实现响应式。

>说白了，这些所谓的状态变量会被js监控，值变了就通过js修改，页面中的值自然会发生变化。

```
export default {
  data() {
    return {
      someObject: {}
    }
  },
  mounted() {
    const newObject = {}
    this.someObject = newObject     

    //两者值一样，但someObject还是状态变量，newObject只是个普通
    console.log(newObject === this.someObject) // false
  }
}
```

## 2 声明方法

`methods` 选项是一个包含所有方法的对象：

```
export default {
  data() {
    return {
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    },
    <!-- increment: () => {
      // 反例：箭头函数无法访问此处的 `this`!
    } -->
  },
  mounted() {
    // 在其他方法或是生命周期中也可以调用方法
    this.increment()
  }
}
```

Vue 自动为 `methods` 中的方法绑定了永远指向组件实例的 `this`。这确保了方法在作为事件监听器或回调函数时始终保持正确的 `this`。不应该在定义 `methods` 时使用箭头函数，因为箭头函数没有自己的 `this` 上下文。

和组件实例上的其他属性一样，方法也可以在模板上被访问。在模板中它们常常被用作事件监听器：

`<button @click="increment">{{ count }}</button>`

## 3 DOM 更新时机

当你更改响应式状态后，DOM 也会自动更新。然而，你得注意 DOM 的更新并不是同步的。相反，Vue 将缓冲它们直到更新周期的 “下个时机” 以确保无论你进行了多少次声明更改，每个组件都只需要更新一次。

可以使用 nextTick() 这个全局 API访问到状态改变后的 DOM：

```
import { nextTick } from 'vue'

export default {
  methods: {
    increment() {
      this.count++
      nextTick(() => {
        // 访问更新后的 DOM
      })
    }
  }
}
```

## 4 默认深层响应式

即使在更改深层次的对象或数组，改动也能被检测到。

```
export default {
  data() {
    return {
      obj: {
        nested: { count: 0 },
        arr: ['foo', 'bar']
      }
    }
  },
  methods: {
    mutateDeeply() {
      // 以下都会按照期望工作
      this.obj.nested.count++
      this.obj.arr.push('baz')
    }
  }
}
```

>也可以直接创建一个[浅层响应式]()对象。它们仅在顶层具有响应性，一般仅在某些特殊场景中需要。

## 5 调用某个函数（而这个函数自身内部有状态）

在某些情况下，我们可能需要动态地创建一个方法函数，比如创建一个预置防抖的事件处理器：

```
import { debounce } from 'lodash-es'

export default {
  methods: {
    // 使用 Lodash 的防抖函数
    click: debounce(function () {
      // ... 对点击的响应 ...
    }, 500)
  }
}
```

不过这种方法对于被重用的组件来说是有问题的，因为这个预置防抖的函数是 有状态的：它在运行时维护着一个内部状态。如果多个组件实例都共享这同一个预置防抖的函数，那么它们之间将会互相影响。

要保持每个组件实例的防抖函数都彼此独立，我们可以改为在 created 生命周期钩子中创建这个预置防抖的函数：

```
export default {
  created() {
    // 每个实例都有了自己的预置防抖的处理函数
    this.debouncedClick = _.debounce(this.click, 500)
  },
  unmounted() {
    // 最好是在组件卸载时
    // 清除掉防抖计时器
    this.debouncedClick.cancel()
  },
  methods: {
    click() {
      // ... 对点击的响应 ...
    }
  }
}
```