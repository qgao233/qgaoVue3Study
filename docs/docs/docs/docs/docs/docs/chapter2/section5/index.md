# v-bind的特殊增强

Vue 专门为 class 和 style 的 v-bind 用法提供了特殊的功能增强：除了字符串外，表达式的值也可以是对象或数组。

## 1 针对class

给`v-bind:class`传递一个对象，判断是否`class="active"`：

```
<div :class="{ active: isActive }"></div>
```

可见`active`是否出现，要依赖于状态变量`isActive`的值。

### 1.1 与class并存

由于`class`的值可以同时有多个，因此vue也支持和一般的 `class`属性共存。

```
data() {
  return {
    isActive: true,
    hasError: false
  }
}

<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

渲染后便是：

```
<div class="static active"></div>   //因为hasError为false，所以text-danger没出现
```

### 1.2 引用对象

```
data() {
  return {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
}

<div :class="classObject"></div>
```

结果一样。

### 1.3 引用计算属性（常用）

```
data() {
  return {
    isActive: true,
    error: null
  }
},
computed: {
  classObject() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}

<div :class="classObject"></div>
```

### 1.4 引用数组

```
data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
}

<div :class="[activeClass, errorClass]"></div>
```

渲染后是：

```
<div class="active text-danger"></div>
```

还可以使用三目表达式：

```
<div :class="[isActive ? activeClass : '', errorClass]"></div>
可以直接换成：
<div :class="[{ active: isActive }, errorClass]"></div>
```

### 1.5 在组件上使用class的情况

一个组件被写好后，通常会被其它组件import后进行使用，那么在被外部使用时会出现两种情况：

* 组件内部只有一个根节点
* 组件内部有多个根节点

#### 1.5.1 一个根节点

声明了一个组件名叫 MyComponent，内部实现的template如下：

```
<!-- MyComponent模板 -->
<p class="foo bar">Hi!</p>
```

被外部使用时添加一些 class：

```
<!-- 在使用组件时 -->
<MyComponent class="baz boo" :class="{ active: isActive }"/>
```

此时这个class的值只会被绑定到唯一的根节点上：

```
<p class="foo bar baz boo active">Hi</p>
```

#### 1.5.1 多个根节点

在实现内部使用`$attrs`获得上层传入的属性，然后根据情况选择在何处使用：

```
<!-- MyComponent 模板使用 $attrs 时 -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>
```

被外部使用时添加一些 class：

```
<MyComponent class="baz" />
```

此时这个class的值传到内部后：

```
<p class="baz">Hi!</p>
<span>This is a child component</span>
```

## 2 针对style

```
data() {
  return {
    activeColor: 'red',
    fontSize: 30
  }
}

<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

CSS的字段支持：

* 驼峰法（`camelCase`）【推荐使用】
* 短横线隔开式 （`kebab-cased`）【对应其 CSS 中的实际名称】

```
<div :style="{ 'font-size': fontSize + 'px' }"></div>
```

### 2.1 引用对象（常用）

```
data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}

<div :style="styleObject"></div>
```

### 2.2 引用计算属性

### 2.3 引用数组

```
<div :style="[styleObject,...]"></div>
```

### 2.4 自动前缀

当你在 `:style` 中使用了需要浏览器特殊前缀的 CSS 属性时，Vue 会自动为他们加上相应的前缀。Vue 是在运行时检查该属性是否支持在当前浏览器中使用。如果浏览器不支持某个属性，那么将测试加上各个浏览器特殊前缀，以找到哪一个是被支持的。

### 2.5 样式多值

你可以对一个样式属性提供多个 (不同前缀的) 值，举例来说：

```
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

数组仅会渲染浏览器支持的最后一个值。在这个示例中，在支持不需要特别前缀的浏览器中都会渲染为 `display: flex`。