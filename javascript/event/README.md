# 事件
## 事件流
定义：表示页面接收事件的顺序

- 事件冒泡
- 事件捕获

### 事件冒泡
定义：表示事件从最具体的元素(文档树最深的节点)开始触发，然后向上传播至没有那么具体的元素(document)

表现形式：从触发了click事件的元素开始，然后click事件沿DOM树一路向上，在经过的节点上依次触发click事件，直达document对象

    所有浏览器都支持事件冒泡，现代浏览器中的事件会一直冒泡到window对象

### 事件捕获
定义：表示最不具体的元素节点最先收到事件，而最具体的节点最后收到事件

表现形式：与事件冒泡刚好是相反的执行顺序

作用：为了事件到达最终目标前拦截事件

    旧版本的浏览器不支持事件捕获，实际开发中不会实现事件捕获，除非特殊情况

### DOM事件流
DOM2规定事件流分为3个阶段：事件捕获、到达目标和事件冒泡，在捕获阶段不会接收到事件，事件会在到达目标阶段触发

    所有现代浏览器都支持DOM事件流(IE8及更早版本不支持)，即在捕获阶段可以会触发事件

## 事件处理程序
事件意味着浏览器或者用户执行的某种动作，如click、load、mouseover，为响应事件而调用的函数成为事件处理程序，又称为事件监听器，通常以on开头，如onclick

### HTML 事件处理程序
事件处理程序，会创建一个函数，该函数有一个特殊的局部变量，保存着`event对象`，且在这个函数中，`this的值相当于事件的目标对象`

#### 作用域链
作用域链被拓展了，通过`with`实现，在该函数中可以访问到document和元素自身的成员(this)

#### 时机问题
场景：HTML元素已经显示在页面上了，用户已经完成交互了，而事件处理程序的代码还无法执行

解决方法：大多数HTML事件处理程序会封装在try / catch 块中

### DOM0 事件处理程序
要使用js指定事件处理程序，必须先取得要操作对象的引用，每个元素通常都有`小写的事件处理程序属性`，比如onclick，只要把这个属性赋值为一个函数即可

#### 作用域
事件处理程序的作用域即为元素的作用域，也就是`this元素`，通过this可以访问到元素的任何属性和方法

#### 移除事件
通过将事件处理程序属性的值设置为`null`，可以移除DOM方式添加的事件处理程序

### DOM2 事件处理程序
#### 赋值 addEventListener
参数：事件名(字符串)、事件处理函数和一个布尔值(事件流顺序)

布尔值：false(默认值)表示冒泡阶段调用事件处理程序

优势：可以为同一个事件添加`多个事件处理函数`(相比DOM0)

    通过addEventListener添加的事件只能使用removeEventListener并传入与添加时相同的参数来移除，因此匿名函数无法移除

## 事件对象
所有浏览器都支持`event对象`

### DOM 事件对象
event对象是传给事件处理程序的唯一参数，不管是onclick还是addEventListener

event对象常用的公共属性和方法：
    
    在事件处理程序内部，this对象始终等于currentTarget的值

- currentTarget：表示当前事件处理程序正在监听的DOM元素，只读
- target：表示当前触发事件的DOM元素，只读
- cancelable：表示是否可以取消事件的默认行为，只读
- type：表示被触发的事件类型，如click
- preventDefault()：用于取消事件的默认行为，只有cancelable为true才可以调用该方法
- stopPropagation()：用于取消所所有后续事件捕获或事件冒泡，只有bubbles为true才可以调用该方法


    event对象只在事件处理程序执行期间存在，一旦执行完毕，就会被销毁

### IE 事件对象

## 事件类型
### 用户界面事件
#### load 事件
定义：表示在整个页面(包括所有外部资源如图片、js文件和css文件)加载完成后触发

##### js方式 指定load事件处理程序
使用addEventListener()方法来指定事件处理程序，该事件也会收到一个event对象
```typescript
window.addEventListener("load", (event) => {
 console.log("Loaded!");
});
```

1、获取图片元素，然后给该元素添加load事件的监听即可

2、与图片不同，要下载js文件必须同时指定src属性并把script元素添加到文档中

3、与script节点相同，link节点也必须指定src属性并把link节点添加到文档中后才会下载样式表

##### html方式 指定load事件处理程序
向<body>元素添加onload属性
```
<body onload="console.log('Loaded!')"></body>
```

1、图片上也会触发load事件，包括DOM中的图片和非DOM中的图片，直接在img标签中添加onload属性指定事件处理程序，表示图片加载完后直接事件处理程序

#### unload 事件
与load事件相对，表示unload事件会在文档卸载完成后触发，一般是在从一个页面导航到另一个页面时触发，常用于清理引用，避免内存泄露，
与load事件类似，unload事件处理程序也有两种指定方式，js和html

    unload事件是在页面卸载完成后触发的，不能使用页面加载后才有的对象，否则会导致错误

#### resize 事件
定义：浏览器窗口被缩放到新高度或宽度时，会触发resize事件

触发方式：

- window监听resize事件
- body元素添加onresize属性


    浏览器窗口在最大化和最小化时也会触发resize事件

#### scroll 事件
虽然scroll事件发生在window上，但实际上反映的是`页面中相应元素`的变化

浏览器的不同模式(`document.compatMode`)，scroll事件获取的方式不同

##### 混杂模式
通过body元素检测scrollTop和scrollLeft属性的变化
```typescript
document.documentElement.scrollTop
```

##### 标准模式
通过html元素检测scrollTop和scrollLeft属性的变化
```typescript
document.body.scrollTop
```

### 焦点事件
页面元素获得或失去焦点时触发

#### 类型

- blur：元素失去焦点时触发，该事件不冒泡，所有浏览器都支持
- focus：元素获取焦点时触发，不冒泡，所有浏览器都支持
- focusin：元素获取焦点时触发，冒泡
- focusout：元素失去焦点时触发，冒泡

#### 焦点转移过程
当焦点从页面中的一个元素移动到另一个元素上，依次发生如下事件：

1、focusout在失去焦点的元素上触发

2、focusin在获取焦点的元素上触发

3、blur在失去焦点的元素上触发

4、focus在获取焦点的元素上触发

    DOMFocusOut和DOMFocusIn在DOM3 Event中被废除了，所以就不加以考虑

### 鼠标和滚轮事件
DOM3 Event定义了9种鼠标事件：

- click：用户单击或键盘回车触发，触发onclick事件处理程序
- dblclick：用户双击触发
- mousedown：在用户按下任意鼠标键时触发，不能通过键盘触发
- mouseenter：用户将鼠标光标从元素`外部`移动到元素`内部`时触发，该事件不冒泡，也不会在光标经过后代元素时触发
- mouseleave：用户将鼠标光标从元素内部移动到元素外部时触发，该事件不冒泡，也不会在光标经过后代元素时触发
- mousemove：用户将鼠标在元素上移动时反复触发
- mouseover：用户将鼠标光标从元素外部移动到元素内部时触发，该事件`支持冒泡`，光标经过`后代元素`时也会触发
- mouseout：用户将鼠标光标从元素内部移动到元素外部时触发，该事件`支持冒泡`，光标经过`后代元素`时也会触发
- mouseup：用户释放鼠标键时触发

#### 鼠标事件顺序
mousedown、mouseup、click、mousedown、mouseup、dblclick

    click事件的触发依赖于mousedown和mouseup事件，而mousedown和mouseup事件不会受其他事件的影响

#### mousewheel 鼠标滚轮事件
##### 客户端坐标
    不包含菜单工具栏客户端视口距离，不会计算滚动条滚动的距离

- clientX：事件被触发时`鼠标指针`相对于浏览器`页面`（或客户区）的水平偏移量
- clientY：事件被触发时`鼠标指针`相对于浏览器`页面`（或客户区）的垂直偏移量

##### 页面坐标
    会加上滚动条滚动的距离，在没有滚动条的情况下与客户端坐标完全相同

- pageX：事件被触发鼠标光标相对于`网页`左上角的水平偏移量
- pageY：事件被触发鼠标光标相对于`网页`左上角的垂直偏移量

##### 屏幕坐标
    鼠标事件不仅是在浏览器窗口触发的，也是在整个屏幕上触发的

- screenX：事件触发鼠标光标相对于`屏幕`左上角的水平偏移量
- screenY：事件触发鼠标光标相对于`屏幕`左上角的垂直偏移量

##### 修饰键
键盘上的修饰键 Shift、Ctrl、Alt 和 Meta 经常用于修改鼠标事件的行为，DOM规定了对应的4个属性来表示这4个按键是否按下，分别是shiftKey、ctrlKey、altKey 和 metaKey

##### 相关元素
DOM通过event对象的 `relatedTarget属性`提供了相关元素的信息。这个属性只有在`mouseover`和`mouseout`事件发生时才包含值，其他所有事件的这个属性的值都是null

### 键盘与输入事件

- keydown：表示用户按下键盘上某个键时触发，持续按住会重复触发
- keyup：用户释放键盘上某个键时触发
- keypress：用户按下键盘上某个键并产生字符时触发，持续按住会重复触发

    
    DOM3 Event废弃了keypress事件，推荐使用textInput事件

#### 键码
event.keyCode：event对象的`keyCode属性`中会保存一个键码，对应键盘上特定的一个键，对于字母和数字键，keyCode的值与`小写字母`和数字的`ASCII编码`一致

#### textInput 事件
表示字符被输入到`可编辑区域`时触发，作为keypress的替代

##### 与keypress的区别

- keypress事件表示可以在任何可以获取到焦点的元素上触发
- textInput事件只有在新字符串被插入时才会触发，keypress对任何能影响文本的键都会触发(包括回退键)

##### 属性

- data：表示要插入的字符(不是字符编码)，shift + r表示R
- inputMethod：表示输入文本的手段，常见的值有1(键盘)、2(粘贴)、3(拖放)、

### 合成事件
DOM3 新增的事件，用于处理IME 输入的复杂输入序列，可以让用户输入物理键盘上没有的字符

#### 类型

- compositionstart：表示在IME的文本系统打开时触发，表示输入即将开始
- compositionupdate，在新字符插入输入字段时触发
- compositionend：表示在IME的文本系统关闭时触发，表示恢复正常键盘输入

### HTML5 事件
#### contextmenu 事件
windows 通过鼠标右键为用户增加了上下文菜单的概念，而mac 通过ctrl + 单击

contextmenu事件冒泡，只要给document指定一个事件处理程序就可以处理页面上的所有同类事件，可以通过`event.preventDefault()`取消事件冒泡

#### beforeunload 事件
给开发者提供阻止页面被卸载的机会，这个事件会在页面即将从浏览器中卸载时触发，会向用户显示一个确认框，请用户确认是希望关闭页面，还是继续留在页面上

设置为要在确认框中显示的字符串:
- event.returnValue属性(IE和Firefox)

- 作为函数值返回(Safari和Chrome)

#### DOMContentLoaded 事件
该事件会在DOM树构建完成后立即触发，不用等待图片、js文件、css文件或其他资源加载完成

#### readystatechange 事件
提供文档或元素加载状态的信息，支持该事件的每个对象都有一个readyState属性

- uninitialized：对象存在并尚未初始化
- loading：对象正在加载数据
- loaded：对象已经加载完数据
- interactive：对象可以交互，但尚未加载完成
- complete：对象加载完成

#### pageshow 与 pagehide 事件
往返缓存：使用浏览器的 前进 和 后退 按钮时加快页面之间的切换，这个缓存不仅存储页面数据，也存储DOM和JS状态，实际上是把整个页面都保存在内存里

    如果页面在缓存中，那么导航到这个页面时就不会触发load事件

##### pageshow 事件
定义：页面显示时触发，无论是否来自往返缓存。在新加载的页面上，pageshow事件会在load事件之后触发；在来自往返缓存的页面上，pageshow事件会在页面状态完全恢复后触发

    这个事件的目标是document，但是事件处理程序必须添加到window上

persisted属性：布尔值，如果页面存储在往返缓存中则返回true，否则返回false

##### pagehide 事件
定义：页面从浏览器中卸载后触发，在unload事件之前触发

persisted属性：页面在卸载之后会被保存在往返缓存中则返回true，因此第一次触发pageshow事件时persisted属性始终都是false，而
第一次触发pagehide事件时persisted属性始终是true；

#### hashchange 事件
表示URL散列值(#后面的部分)发生变化时通知开发者

属性：oldURL和newURL分别保存变化前后的URL，而且是包含散列值完整URL

    如果想确定当前的散列值，最好使用location对象，loaction.hash

## 内存和性能
### 事件委托
定义：利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类的所有事件

场景：每当事件处理程序指定给元素时，运行中的浏览器代码就会和js代码之间建立连接，连接越多，页面执行速度就越慢

使用：只需要在DOM树中尽量最高的层次上添加一个事件处理程序，最高层次下的所有子节点，都会被这个事件处理程序处理

好处：减少了对DOM元素的操作，占用的内存更少

### 移除事件处理程序
场景：对于内存中留有的过时不用了的`空事件处理程序`，需要及时移除，它们也会影响内存和性能问题

    如果页面在卸载之前没有及时清理事件处理程序，它们就会滞留在内存中，每次加载完页面再卸载页面，内存中滞留的对象数目就会增加
    加载完页面再卸载页：可能是两个页面间来回切换，也可能是刷新页面

使用：通常使用innerHTML替换页面中的某一部分

    一个元素被移除的同时需要手动移除该元素的事件处理程序
    如：btn.onlick = null
    通常在设置innerHTML之前，先移除事件处理程序，确保内存可以被再次利用

## 模拟事件
### DOM中的事件模拟
1、在`创建`了event对象之后

2、还需要使用与事件有关的信息对其进行`初始化`，具体的信息与createEvent方法中使用的参数有关

2、模拟事件还需要调用dispatchEvent()方法`触发`事件

#### 模拟鼠标事件
创建鼠标事件对象的方法是为createEvent()传入字符串`MouseEvent`，返回的对象有一个名叫`initMouseEvent`的方法，
用于指定与该鼠标事件有关的信息

initMouseEvent参数(重要的)：

- type：字符串，表示要触发的事件类型
- bubbles：布尔值，表示事件是否应该冒泡
- cancelable：布尔值，表示事件是否可以取消
- view：与事件关联的视图，基本上设置为document.defaultView

#### 模拟键盘事件
createEvent()传入的字符串是`KeyboardEvent`，返回的对象包含一个`initKeyEvent`的方法，参数与上述一致

#### 模拟其他事件
模拟变动事件和HTMl事件

##### 变动事件
createEvent()传入的字符串是`MutationEvents`，返回的对象包含一个`initMutationEvent`的方法，参数与上述一致

可以用与模拟DOM节点的变动情况，如插入`DOMNodeInserted`、删除、添加

##### HTML事件
createEvent()传入的字符串是`HTMLEvents`，返回的对象包含一个`initEvent`的方法，参数与上述一致

#### 自定义DOM事件
createEvent()传入的字符串是`CustomEvent`，返回的对象包含一个`initCustomEvent`的方法

initCustomEvent参数：

- type：字符串，触发事件的类型
- bubbles：布尔值，表示事件是否冒泡
- cancelable：布尔值，表示事件是否可以被取消
- detail：任意值，保存在event对象的detail属性中