# 事件处理

可以使用 v-on 指令 (简写为 @) 来监听 DOM 事件（click之类的）。

## 1 事件处理器的分类

* 内联事件处理器：直接在双引号里写js语句。

```
<button @click="count++">Add 1</button>
<p>Count is: { { count } }</p>

methods: {
  say(message) {
    alert(message)
  }
}
<button @click="say('hello')">Say hello</button>
```

* 方法事件处理器：一个指向组件上定义的方法的**属性名**或是**路径**。

```
methods: {
  greet(event) {
    // 方法中的 `this` 指向当前活跃的组件实例
    alert(`Hello ${this.name}!`)
    // `event` 是 DOM 原生事件
    if (event) {
      alert(event.target.tagName)   //访问到该 DOM 元素。
    }
  }
}
<!-- `greet` 是上面定义过的方法名 -->
<button @click="greet">Greet</button>
```

从例子可以看出，它只是写了一个方法名字，而不像js调用方法，需要在后面加括号。

### 1.1 区分两种事件处理器

* 方法事件处理器：`foo`、`foo.bar` 和 `foo['bar']` ，
* 内联事件处理器：`foo()` 和 `count++`。

## 2 内联事件处理器中访问event参数

* 向处理器方法传入一个特殊的 $event 变量，
* 或者使用内联箭头函数。

```
methods: {
  warn(message, event) {
    // 这里可以访问 DOM 原生事件
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}

<!-- 使用特殊的 $event 变量 -->
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>

<!-- 使用内联箭头函数 -->
<button @click="(event) => warn('Form cannot be submitted yet.', event)">
  Submit
</button>
```

## 3 修饰符

修饰符是用 `.` 表示的指令后缀。

### 3.1 事件修饰符

在处理事件时调用 event.preventDefault() 或 event.stopPropagation() 是很常见的。尽管我们可以直接在方法内调用，但如果方法能更专注于数据逻辑而不用去处理 DOM 事件的细节会更好。

为解决这一问题，Vue 为 v-on 提供了事件修饰符。包含以下这些：

* .stop
* .prevent
* .self
* .capture
* .once
* .passive

```
<!-- 单击事件将停止传递 -->
<a @click.stop="doThis"></a>

<!-- 提交事件将不再重新加载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰语可以使用链式书写 -->
<a @click.stop.prevent="doThat"></a>

<!-- 也可以只有修饰符 -->
<form @submit.prevent></form>

<!-- 仅当 event.target 是元素本身时才会触发事件处理器 -->
<!-- 例如：事件处理器不来自子元素 -->
<div @click.self="doThat">...</div>
```

>使用修饰符时需要注意调用顺序，因为相关代码是以相同的顺序生成的。因此使用 @click.prevent.self 会阻止元素及其子元素的所有点击事件的默认行为而 @click.self.prevent 则只会阻止对元素本身的点击事件的默认行为。

.capture、.once 和 .passive 修饰符与原生 addEventListener 事件相对应：

```
<!-- 添加事件监听器时，使用 `capture` 捕获模式 -->
<!-- 例如：指向内部元素的事件，在被内部元素处理前，先被外部处理 -->
<div @click.capture="doThis">...</div>

<!-- 点击事件最多被触发一次 -->
<a @click.once="doThis"></a>

<!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
<!-- 以防其中包含 `event.preventDefault()` -->
<div @scroll.passive="onScroll">...</div>
```

.passive 修饰符一般用于触摸事件的监听器，可以用来改善移动端设备的滚屏性能。


>请勿同时使用 .passive 和 .prevent，因为 .passive 已经向浏览器表明了你不想阻止事件的默认行为。如果你这么做了，则 .prevent 会被忽略，并且浏览器会抛出警告。

### 3.2 按键修饰符

在监听键盘事件时，我们经常需要检查特定的按键。Vue 允许在 v-on 或 @ 监听按键事件时添加按键修饰符。

```
<!-- 仅在 `key` 为 `Enter` 时调用 `vm.submit()` -->
<input @keyup.enter="submit" />
```

你可以直接使用 KeyboardEvent.key 暴露的按键名称作为修饰符，但需要转为 kebab-case 形式。

```
<input @keyup.page-down="onPageDown" />
在上面的例子中，仅会在 $event.key 为 'PageDown' 时调用事件处理。
```

#### 3.2.1 vue提供的默认别名

Vue 为一些常用的按键提供了别名：

* .enter
* .tab
* .delete (捕获“Delete”和“Backspace”两个按键)
* .esc
* .space
* .up
* .down
* .left
* .right

### 3.3 鼠标或键盘事件监听器（系统修饰符）

* .ctrl
* .alt
* .shift
* .meta


>在 Mac 键盘上，meta 是 Command 键 (⌘)。在 Windows 键盘上，meta 键是 Windows 键 (⊞)。在 Sun 微机系统键盘上，meta 是钻石键 (◆)。在某些键盘上，特别是 MIT 和 Lisp 机器的键盘及其后代版本的键盘，如 Knight 键盘，space-cadet 键盘，meta 都被标记为“META”。在 Symbolics 键盘上，meta 也被标识为“META”或“Meta”。

举例来说：

```
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + 点击 -->
<div @click.ctrl="doSomething">Do something</div>
```

>请注意，系统按键修饰符和常规按键不同。与 keyup 事件一起使用时，该按键必须在事件发出时处于按下状态。换句话说，keyup.ctrl 只会在你仍然按住 ctrl 但松开了另一个键时被触发。若你单独松开 ctrl 键将不会触发。

### 3.4 `.exact` 修饰符

.exact 修饰符允许控制触发一个事件所需的确定组合的系统按键修饰符。

```
<!-- 当按下 Ctrl 时，即使同时按下 Alt 或 Shift 也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 仅当按下 Ctrl 且未按任何其他键时才会触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 仅当没有按下任何系统按键时触发 -->
<button @click.exact="onClick">A</button>
```

### 3.5 鼠标按键修饰符

* .left
* .right
* .middle

这些修饰符将处理程序限定为由特定鼠标按键触发的事件。