# 表单脚本
## 表单基础
在html中以<form>标签表示，在js中以HTMLFormElement类型表示，该类型继承HTMLElement类型，拥有其它HTML元素一样的默认属性

### HTMLFormElement属性和方法

- acceptCharset：服务器可以接收的字符集
- action：请求的URL
- element：表单中所有控件的HTMLCollection
- enctype：请求的编码类型
- length：表单中的控件数量
- method：请求的方法类型
- name：表单的名字
- reset()：表单字段重置为各自的默认值
- submit()：提交表单


    document.forms集合可以获取到页面上所有的表单元素

### 提交表单
提交按钮可以使用`type属性为submit`的<input>或<button>元素定义

event.preventDefault()可以阻止表单提交

    通过submit方法提交表单不会触发submit事件

### 重置表单
重置表单可以使用`type属性为reset`的<input>或<button>元素定义，所有表单字段都会重置回到第一次渲染时各自拥有的值，
如果字段为空就会变成空，如果字段有默认值，则恢复为默认值

    调用reset方法重置表单会触发reset事件

### 表单字段
所有表单元素都是表单element属性(集合)中包含的一个值，这个集合是一个有序列表，包含对表单中所有字段的引用，可以通过索引位置和name属性访问

#### 表单字段的属性

- disabled：布尔值，表示表单字段是否禁用
- form：指针，指向表单字段所属的表单，只读
- name：字符串，字段名
- readOnly：布尔值，表示字段是否可读
- tabIndex：数值，表示在按Tab键时的切换顺序
- type：字符串，表示字段类型
- value：要提交给服务器的字段值

#### 表单字段的方法

- focus()：使元素获得焦点
- blur()：从元素上移除焦点，焦点不会转移到任何特定元素上

#### 表单字段的事件

- blur：在字段失去焦点时触发
- change：input和textarea元素的value发生变化且失去焦点时触发，select选中项发生变化时触发
- focus：在字段获取焦点时触发

## 文本框编程
input和textarea两种类型的文本框都会在value属性中保存自己的内容，通过这个属性可以读取也可以修改文本内容

    使用value属性，而不是标准DOM方法读写文本框的值

### 选择文本
select()方法：用于全部选中文本框中的文本，调用该方法后自动将焦点设置到文本框

#### select事件
定义：当选中文本框中的文本时，会触发select事件(常规的浏览器中，事件会在用户选择完文本后立即触发)

    调用select方法也会触发select事件

#### 取得选中文本
HTML5新增了两个属性：`selectionStart`和`selectionEnd`，分别表示文本选区的起点和终点

```typescript
function getSelectedText(textbox){
 return textbox.value.substring(textbox.selectionStart, textbox.selectionEnd);
} 
```

#### 部分选中文本
HTML5新增setSelectionRange()方法，接收两个参数：要选择的第一个字符的索引和停止选择的字符的索引(`不包含停止索引处的字符`)

    想要看到选中的效果，则必须让文本框获得焦点

### 输入过滤
#### 屏蔽字符
`keypress事件`负责向文本框插入字符，可以通过阻止这个事件的默认行为来屏蔽字符

如果想只屏蔽特定字符，则需要检查事件的`charCode`属性，需要保留上下箭头键、退格键和删除键时触发keypress事件

    在Firefox中，所有触发keypress事件的非字符键的charCode都是0，而在Safari中，这些键都是8

处理复制、粘贴及涉及Ctrl键的其他功能，`!event.ctrlKey`检测确保没有按下Ctrl键

#### 处理剪贴板
相关事件：

- beforecopy：复制操作发生前触发
- copy：复制操作发生时触发
- beforecut：剪切操作发生前触发
- cut：剪切操作发生时触发
- beforepaste：粘贴操作发生前触发
- paste：粘贴操作发生时触发

剪贴板上的数据可以通过`window对象`(IE)或`event`(Firefox、Safari、Chrome)上的`clipboardData`对象获取

clipboardData对象方法：

- getData()：从剪贴板检索字符串数据，并接收一个参数，该参数是要检索的数据的格式：text和URL(IE)、MIME(Firefox、Safari、Chrome)
- setData()：第一个参数用于指定数据类型，第二个参数是要放到剪贴板上的文本
- clearData()

### HTML5 约束验证

#### 必填字段
给表单字段添加`required属性`，任何带有required属性的字段都必须有值，否则无法提交表单

    浏览器的差异：Safari不会阻止提交表单，其他浏览器会阻止表单提交并显示相关信息提示

#### 更多输入类型
新增了type值：email和url

- email类型：确保输入的文本匹配电子邮箱地址(文本出现@会被认为是有效的电子邮箱地址)
- url类型：确保输入的文本匹配URL

#### 数值范围
新增几种输入元素类型，都是期待某种数值输入的：number、range、datetime、datetime-local、date、month、week、time

对于上述每种数值类型，都可以指定min属性、max属性，以及step属性

方法：

- stepUp()：从当前值加上多少数值，默认步长值会递增1
- stepDown()：从当前值减去多少数值，默认步长值会递减1

#### 输入模式
新增`pattern属性`，这个属性用于指定一个正则表达式，表示用户输入的文本必须与之匹配

#### 检测有效性
使用`checkValidity()方法`可以检测表单中任意给定字段是否有效，有效则返回true(字段值不匹配pattern属性也会被视为无效)

`validity属性`会告诉字段为什么有效或无效，包含一系列返回布尔值的属性：

- patternMismatch：字段值是否匹配指定的pattern
- rangeOverflow：字段值是否大于max
- rangeUnderflow：字段值是否小于min
- stepMisMatch：字段值与min、max、step的值是否相符
- typeMismatch：判断字段值是否为email和url要求的格式
- valueMissing：如果字段是必填的但是没有值返回true

#### 禁用验证
指定`novalidate属性`可以禁止对表单进行任何验证

## 选择框编程
select标签，HTMLSelectElement类型属性和方法：

- add(newOption,relOption)：在relOption之前向组件中添加新的option
- multiple：布尔值，表示是否允许多选
- options：控件中所有option元素
- remove(index)：移除给定位置的选项
- selectedIndex：选中项基于0的`索引值`，没有选中则为-1，对于多选列表，始终是第一个选项的索引
- size：控件中可见的行数


    1、没有选中值，则选择框的值是空字符串
    2、选中项其value属性没有指定值，则选择框的值是该项的文本内容

option标签，HTMLOptionElement类型的属性和方法：

- index：选项在options集合中的索引
- label：选项的标签
- selected：布尔值，表示是否选中了当前项
- text：表示选项的文本
- value：选项的值


    其他表单字段会在自己的值改变后出发change事件，然后字段失去焦点，选择框会在选中一项时立即触发change事件

### 选项处理
1、对于`只允许选择一项`的选择框，通过`selectedIndex属性`可以获取选项，并且可以获取关于选项的所有信息

2、默认选中选择项，通过`selected属性`为true表示`选中`

    如果修改单选框中选项的selected属性，则其他选项会被移除；修改多选框不会移除其他选项，可以动态选择任意多个选项

### 添加选项
#### DOM
使用js动态创建选项并将它们添加到选择框中
```typescript
let newOption = document.createElement("option");
newOption.appendChild(document.createTextNode("Option text"));
newOption.setAttribute("value", "Option value");
selectbox.appendChild(newOption); 
```

#### Option 构造函数
使用Option构造函数创建新的选项，Option函数接收两个参数：text和value，其中value是可选的
```typescript
let newOption = new Option("Option text", "Option value");
selectbox.appendChild(newOption);
```

#### add 方法
使用选择框的add()方法，接收两个参数：要添加的新选项和要添加到其前面的参考选项

    如果要在末尾添加选项，则第二个参数是null
```typescript
let newOption = new Option("Option text", "Option value");
selectbox.add(newOption, undefined); // 最佳方案
```

### 移除选项
#### DOM
使用DOM的removeChild()方法并传入要移除的选项
```typescript
selectbox.removeChild(selectbox.options[0]); // 移除第一项
```

#### remove 方法
使用选择框的remove()方法，接收一个参数：要移除选项的索引
```typescript
selectbox.remove(0); // 移除第一项
```

#### null
直接将选项设置为null
```typescript
selectbox.options[0] = null; // 移除第一项
```

#### 移除所有选项
使用迭代所有选项并逐一移除它们，移除第一项会自动将所有选项往前移一位
```typescript
function clearSelectbox(selectbox) {
 for (let option of selectbox.options) {
 selectbox.remove(0);
 }
} 
```

### 移动和重排选项
使用appendChild()方法可以直接将选项从第一个选择框移动到第二个选择框，常用于将选项移动到最后

使用insertBefore()方法可以将选项移动到选择框中的特定位置

## 富文本编辑
定义：将空白文档变成`可编辑`的，实际编辑的是body元素的HTML

技术点：在空白HTML文件中嵌入一个iframe，通过`designMode属性`设置为on，整个文档会变成可编辑的(显示插入光标)

    只有在文档完全加载之后才可以设置designMode属性，需要使用onload事件处理程序

### contentEditable
给页面中的任何元素指定contenteditable属性，然后该元素会立即被用户编辑，不需要额外的iframe、空页面和js

通过设置contentEditable属性，可以随时切换元素的可编辑状态

### 与富文本交互
