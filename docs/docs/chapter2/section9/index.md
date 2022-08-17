# 表单输入绑定

在前端处理表单时，我们常常需要将表单输入框的内容同步给 JavaScript 中相应的变量。

手动连接值绑定和更改事件监听器可能会很麻烦：

```
<input
  :value="text"
  @input="event => text = event.target.value">
```

## 1 v-model（静态的字符串）

v-model 指令帮我们简化了这一步骤：

```
<input v-model="text">
```

v-model 支持：

* `<input>` 和 `<textarea>` ：绑定 value 属性并侦听 input 事件；
* `<input type="checkbox/radio">` ：绑定 checked 属性并侦听 change 事件；
* `<select>` ：绑定 value 属性并侦听 change 事件：

>使用v-model 会忽略任何表单元素上初始的 value、checked 或 selected 属性。

### 1.1 textarea

```
<!-- 错误 textarea不支持插值表达式-->
<textarea>{ { text } }</textarea>

<!-- 正确 -->
<textarea v-model="text"></textarea>
```

### 1.2 checkbox

```
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{ { checked } }</label>
```

将多个复选框绑定到同一个数组或集合的值：

```
export default {
  data() {
    return {
      checkedNames: []
    }
  }
}
<div>Checked names: { { checkedNames } }</div>

<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>

<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>

<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
```

### 1.3 radio

```
<div>Picked: { { picked } }</div>

<input type="radio" id="one" value="One" v-model="picked" />
<label for="one">One</label>

<input type="radio" id="two" value="Two" v-model="picked" />
<label for="two">Two</label>
```

### 1.4 select

```
<div>Selected: { { selected } }</div>

<select v-model="selected">
  <option disabled value="">Please select one</option>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

>如果 v-model 表达式的初始值不匹配任何一个选择项，`<select>` 元素会渲染成一个“未选择”的状态。在 iOS 上，这将导致用户无法选择第一项，因为 iOS 在这种情况下不会触发一个 change 事件。因此，我们建议提供一个空值的禁用选项，如上面的例子所示。

多选 (值绑定到一个数组)：

```
<div>Selected: { { selected } }</div>

<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
```

**v-for 动态渲染选项**

```
export default {
  data() {
    return {
      selected: 'A',
      options: [
        { text: 'One', value: 'A' },
        { text: 'Two', value: 'B' },
        { text: 'Three', value: 'C' }
      ]
    }
  }
}
<select v-model="selected">
  <option v-for="option in options" :value="option.value">
    { { option.text } }
  </option>
</select>

<div>Selected: { { selected } }</div>
```

## 2 v-model（动态数据）

### 2.1 checkbox

```
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no" />
```

`true-value` 和 `false-value` 是 Vue 特有的属性，仅支持和 v-model 配套使用。这里 toggle 属性的值会在选中时被设为 'yes'，取消选择时设为 'no'。你同样可以通过 v-bind 将其绑定为其他动态值：

```
<input
  type="checkbox"
  v-model="toggle"
  :true-value="dynamicTrueValue"
  :false-value="dynamicFalseValue" />
```

>true-value 和 false-value attributes 不会影响 value attribute，因为浏览器在表单提交时，并不会包含未选择的复选框。如果要保证这两个值 (例如：“yes”和“no”) 的其中之一被表单提交，请使用单选按钮作为替代。

### 2.2 radio

```
<input type="radio" v-model="pick" :value="first" />
<input type="radio" v-model="pick" :value="second" />
```

pick 会在第一个按钮选中时被设为 first，在第二个按钮选中时被设为 second。

### 2.3 select

```
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option :value="{ number: 123 }">123</option>
</select>
```

v-model 同样也支持非字符串类型的值绑定！在上面这个例子中，当某个选项被选中，selected 会被设为该对象字面量值 { number: 123 }。

## 3 修饰符

### 3.1 .lazy

默认情况下，v-model 会在每次 input 事件后更新数据 (IME 拼字阶段的状态例外)。你可以添加 lazy 修饰符来改为在每次 change 事件后更新数据：

```
<!-- 在 "change" 事件后同步更新而不是 "input" -->
<input v-model.lazy="msg" />
```

>input输入框的onchange事件，要在 input 失去焦点的时候才会触发。

### 3.2 .number

如果你想让用户输入自动转换为数字，你可以在 v-model 后添加 .number 修饰符来管理输入：

```
<input v-model.number="age" />
```

如果该值无法被 parseFloat() 处理，那么将返回原始值。

>number 修饰符会在输入框有 type="number" 时自动启用。

### 3.3 .trim

如果你想要默认自动去除用户输入内容中两端的空格，你可以在 v-model 后添加 .trim 修饰符：

`<input v-model.trim="msg" />`

## 4 [组件上的 v-model](https://staging-cn.vuejs.org/guide/components/events.html#usage-with-v-model)