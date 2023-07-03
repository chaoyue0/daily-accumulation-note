# JavaScript

## 事件循环和消息队列
定义：工作线程将消息放到消息队列中，主线程通过事件循环过程去取消息

- 消息队列：是一个先进先出的队列，它里面存放着各种信息
- 事件循环：主线程重复从消息队列中取信息、执行的过程

注：异步过程的回调函数，一定不在当前这一轮的事件循环中

## 函数表达式
### 执行环境

定义：包含当前正在执行的代码、所使用的变量和函数，以及相关的作用域信息。可以理解为代码执行的上下文，
决定`代码`在运行过程中的`行为和访问的资源`

变量对象：与执行环境一一对应，环境中定义的所有变量和函数都定义在这个对象中；

活动对象：执行环境为`函数`的变量对象

#### 作用域

定义：代码中定义`变量`的可访问范围，规定了在何处以及如何查找变量，决定了`变量的可见性和生命周期`

#### 作用域链

定义：当前执行环境的变量对象和所有外部环境的变量对象组成的链式结构

作用：保证对环境有权访问的所有变量和函数的`有序访问`

### 垃圾回收
在js中，所需内存的分配以及无用内存的回收完全实现了`自动管理`，对于`不可达`的内存会被垃圾回收机制清除

可达性：以某种方式可以访问或者可以用到的值，一定是存储在内存中的

孤岛：一组相互引用的对象，但是外部没有对其他对象的引用，这组对象也是不可达的，会从内存中被删除

#### 标记清除

当变量进入环境，给变量标记为"进入环境"，从逻辑上讲，不能释放进入环境的变量所占用的内存。而当变量离开环境时，
则将其标记为"离开环境"

#### 引用计数

当声明了一个变量并将一个引用类型值赋给该变量时，这个变量的引用次数就是1，如果该变量又被赋给另一个变量，则
这个变量的引用次数再加1。如果这个引用次数变成0，那么就说明无法访问到这个值，因此就可以被垃圾回收机制回收；

### 闭包
定义：指的是函数与其词法环境的组合，及闭包是指有权访问另一个函数作用域中的变量的函数

创建闭包：在以函数内部创建另一个函数并返回

#### 函数调用
当某个函数被调用时，会创建一个`执行环境`以及相应的`作用域链`，并且作用域链上包含使用argument和其他命名参数的值来初始化函数的`活动对象`，
外部函数的活动对象处于次级位置，依此类推直至作用域终点的全局执行环境

#### 函数参数arguments
定义：对应函数参数的类数组对象

特点：

- 除了length属性和索引元素访问之外没有任何Array属性
- typeof类型判断返回Object;Object.prototype.toString.call()返回[ object Arguments ]

属性：

- callee：指向参数所属的当前执行的函数，常表示匿名函数，`现在已被禁用`
- length：传递给函数的参数数量

#### 函数属性[[scopes]]
定义：属于函数的内部属性，无法访问到。存储的执行上下文对象的集合，该集合呈链式链接，即作用域链

顺序：函数内部访问的对象先从函数自己的作用域内部寻找，没有找到则从该函数的外层寻找，Global全局作用域属于最外层

#### 闭包的作用

- 可以读取函数内部的变量
- 让这些值始终保持在内存中

#### 闭包的特性

- 访问外部函数的变量：即使外部函数已经执行完毕
- 保护变量：闭包可以创建一个私有作用域，外部函数无法访问内部函数的变量，可以有效的保护变量，防止被外部访问和修改
- 延长变量的生命周期：闭包持有对外部函数作用域的引用，外部函数变量不会在函数执行完毕后被销毁

#### 闭包的解决方案

- 立即执行函数

会创建一个独立的作用域，通过使用立即执行函数，可以避免将变量泄露到全局作用域，同时可以创建一个私有的作用域
，来确保内部变量在函数外部不可访问

### 私有变量
在js中没有私有成员的概念，所有对象都是公有的。但是存在一个私有变量的概念，在函数中定义的变量可以认为是私有变量，包括函数参数、局部变量和函数内部定义的其他函数

特权方法：在函数内部创建一个闭包，闭包通过自己的作用域链可以访问到定义在函数内部的变量，因此可以创建用于访问私有变量的公共方法，即特权方法

#### ES6
class类属性在默认情况下是公有的，可以增加哈希前缀#的方法来定义私有类字段，#也是名称的一部分，声明和访问时也需要带上。

在作用域外引用#名称、内部未声明的情况下引用私有变量、尝试使用delete移除声明的字段都会报错
```allykeynamelanguage
class ClassWithPrivateField {
  #privateField;

  constructor() {
    this.#privateField = 42;
    delete this.#privateField;   // 语法错误
    this.#undeclaredField = 444; // 语法错误
  }
}

const instance = new ClassWithPrivateField()
instance.#privateField === 42;   // 语法错误
```

    可以使用in运算符检查私有字段是否存在，当私有字段存在时，运算符返回true，否则返回false

#### 闭包
在构造函数内部使用闭包，将私有变量和私有方法封装在类的实例中，只有类的公共内部方法可以访问这些私有变量，在类的外部无法直接访问到私有变量
```allykeynamelanguage
class MyClass {
  constructor() {
    // 私有变量
    let privateVariable = 'private';

    // 私有方法
    let privateMethod = function() {
      console.log('Private method');
    };

    // 公共方法可以访问私有变量和私有方法
    this.publicMethod = function() {
      console.log('Public method');
      console.log(privateVariable);
      privateMethod();
    };
  }
}
```

#### TypeScript
在Ts中有访问修饰符`private`来声明私有变量和方法，保证只能在类的内部可以访问私有成员，外部无法直接访问


#### Symbol
symbol()函数返回的symbol值是唯一的，且能够作为对象属性的唯一标示符，可以用来实现类私有变量
```allykeynamelanguage
const MyObject = (function() {
     let _value = Symbol('value')
     let _value2 = Symbol('value')
     _value == _value2   // false
     
     class MyObject {
         constructor() {
             this[_value] = 123
         }
         
         getValue() {
             return this[_value]
         }

         setValue(newValue) {
             this[_value] = newValue
         }
     }
     
     return MyObject
 })()
```

#### 模块模式
为单例创建私有变量和特权方法，所谓单例，指的就是只有一个实例的对象，以`对象字面量`的方式来创建单例对象

```allykeynamelanguage
const singleton = function() {
let value = 123

return {
// 其他公有属性
getValue() {
return value
},
setValue(newValue) {
value = newValue
}
}
}()
```

## BOM
### Window对象
#### frame框架
```allykeynamelanguage
<frameset cols="50%,50%">
   <frame src="red.htm" name="redFrame">
   <frame src="blue.htm" name="blueFrame">
</frameset>
```

- 页面包含多个框架的情况下，每个框架都有自己的window对象或Global对象
- top对象始终指向最高曾的框架
- 不同frame之间使用postMessage方法实现跨域通信
- 不同frame都有自己的渲染上下文，但实际上在同一个主线程中执行

#### 窗口
##### 窗口距离屏幕的偏移量

- window.screenLeft：表示窗口左边缘距离屏幕左边缘的偏移量(火狐浏览器不支持)
- window.screenTop：表示窗口顶部边缘距离屏幕顶部边缘的偏移量(火狐浏览器不支持)
- window.screenX：(IE浏览器不支持)
- window.screenY：(IE浏览器不支持)

```allykeynamelanguage
如果外接显示器，存在有个显示器的情况，需要考虑多个显示器的布局位置
```

##### 窗口的偏移量

- window.moveTo(x,y)：将窗口移动到(x,y)处
- window.moveBy(x,y)：将窗口在横坐标移动x，纵坐标移动y

##### 窗口的大小
- window.innerWidth：表示浏览器窗口内部可见区域的宽度
- window.innerHeight：表示浏览器窗口内部可见区域的高度
- window.outerWidth：表示浏览器窗口的宽度，包括边框、滚动条等
- window.outerHeight：表示浏览器窗口的高度，包括边框、滚动条等

##### 窗口大小的调整
- window.resizeTo(a,b)：将窗口的大小调整到a x b尺寸
- window.resizeBy(a,b)：在现有窗口的大小下将窗口的大小扩大或者缩小

##### 窗口弹出
window.open(url,窗口目标,特性字符串)





#### 间歇调用和超时调用
原理：定时器属于异步任务，定时器里面的执行代码属于异步代码，这里的代码不会立即执行，需要等待队列为空，并达到规定的延时时间才会执行

##### 超时调用
定义：表示指定时间后执行代码

setTimeout(要执行的代码,毫秒时间),调用setTimeout之后，该方法会返回一个数值ID，该ID是执行代码唯一标识符，通过它来取消超时调用
调用clearTimeout方法，将ID作为参数

##### 间歇调用
定义：表示每隔指定的时间就会执行一次代码

setInterval()，参数与setTimeout相同

