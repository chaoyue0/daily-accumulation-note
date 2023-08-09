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