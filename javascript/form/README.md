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