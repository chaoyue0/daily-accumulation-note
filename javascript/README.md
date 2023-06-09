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

#### 系统对话框
- alert('提示性文字')
- confirm('提示性文字'),返回布尔值，表示用户点击的是确认还是取消
- prompt('文本提示','文本框默认值'),点击取消返回null
- window.print()：显示打印对话框，异步显示的，对话框计数器不会将他们计算在内，因此不会受到用户禁用后续对话框显示的影响
- window.find()：显示查找对话框，也是异步显示的

### location对象
window和document的location属性引用的都是同一个对象，可以访问到URL

- hash：返回hash值，#号后跟着的字符串
- host：返回服务器名称和端口号
- port：返回端口号
- hostname：返回服务器名称
- href：返回完成的URL
- protocol：返回协议
- search：返回URL的查询字符串，以问号开头

注：每次修改location属性后(hash除外)，页面都会以新URL重新加载

#### 位置操作
前进：location.assign(url)：立即打开新URL并在浏览器的历史记录中生成一条记录，可以通过点击后退导航到前一个页面

    window.location = url 和 location.href = url 与上述API具有相同的效果

前进：如果要禁用"后退"行为，可以用location.replace(url)方法，用户不能回到前一个页面

加载：location.reload()，重新加载当前显示的页面，不传参数可能从缓存中加载，传递参数true表示从服务器重新加载

### navigator对象
常用于检测网页的浏览器类型

#### 检测插件
navigator.plugins

### screen对象

- availHeight：减去系统部件高度之后的值，表示可用高度，只读
- availWidth：减去系统部件宽度之后的值，表示可用宽度，只读
- 
### history对象

- go(x)：正数表示前进，负数表示后退，字符串表示跳转到最近的特定页面
- back()：表示后退一页
- forward()：表示前进一页
- length属性：表示所有历史记录，第一个页面length等于0

## 客户端检测
不同浏览器兼容性检测JS库：Modernizr

### 能力检测
目标：不是识别特定的浏览器，而是识别浏览器的能力

1、先检测达成目的的最常用的特性，保证代码最优化，避免测试多个条件

2、必须测试实际要用到的特性，一个特性存在，不一定意味着另一个特性也存在

#### 更可靠的能力检测
使用typeof判断是否为一个函数

### 怪癖检测
定义：识别浏览器的特殊行为，想要知道浏览器存在什么缺陷(bug)

实现：需要运行一小段代码，以确定某个特性不能正常工作

### 用户代理检测
定义：检测用户代理字符串来确定实际使用的浏览器，navigator.userAgent属性访问

## DOM
### DOM1
#### Node
定义：js中所有节点类型都继承自Node类型，因此所有的节点类型都共享相同的属性和方法

##### NodeType节点类型
12个数值常量表示或Node的12个常量属性，常用的是元素节点和文本节点

##### 节点信息

- nodeName：保存的始终是元素的标签名
- nodeValue：始终是null

##### 节点关系
节点之间的关系可以用传统的家族关系来描述，一种树形结构

孩子节点：每个节点都有一个`childNodes属性`，属性值是一个NodeList类数组对象，用于保存一组有序的节点，可以通过位置来访问这些节点

访问方式：

- 通过类似数组的方法，使用下标加[]的方式
- 使用item(x)方法，其中x表示下标位置

父节点：每个节点都有一个`parentNode属性`，该属性指向文档树中的父节点

同胞节点：包含在childNodes列表中的每个节点之间互相之间都是同胞节点，通过`previousSibling属性`和`nextSibling属性`可以访问到其他同胞节点

##### 操作节点

- appendChild：用于向childNode列表的末尾添加一个节点，返回新增的节点
- insertBefore：在列表的指定位置插入节点，接受两个参数，插入的节点和插入的位置
- replaceChild：替换节点，接受两个参数，插入的节点和要替换的节点，被替换的节点仍然在文档中，但是在文档中已经没有自己的位置了
- removeChild：接受一个参数，就是要移除的节点，返回该移除节点
- cloneNode：接受一个布尔值参数，表示是否执行深复制，深复制就是复制节点及其整个子节点树，复制后返回的节点副本属于文档所有，并没有为它指定父节点
- normalize：无参数，表示将当前节点和它的后代节点"规范化"，不存在一个空的文本节点，或者两个相邻的文

#### Document
document对象是继承自Document类型的是一个实例，表示整个HTML页面，属于window对象的一个属性

特点：

- nodeType值为9
- nodeName值为'#document'
- nodeValue值为null
- parentNode值为null

##### 文档的子节点

- documentElement属性：可以访问到HTML页面中的html元素
- childNode列表：可以通过下标索引访问到文档元素
- body：访问到body元素

##### 文档信息

- title：页面的标题
- url：地址栏中显示的url地址
- domain：只包含页面的域名，可以修改，但是不能将这个属性设置为url中不包含的域。有一个限制，一开始是'松散的'就不能再设置为'紧绷的'，否则会导致错误

```allykeynamelanguage
//假设页面来自于 p2p.wrox.com 域
document.domain = "wrox.com"; //松散的(成功)
document.domain = "p2p.wrox.com"; //紧绷的(出错!)

```
- referrer：保存链接到当前页面的那个源页面的URL地址，没有源页面的情况下，返回空字符串

##### 查找元素

- getElementById：接受一个参数，元素的id。如果多个元素的id相同，则返回文档中第一次出现的元素
- getElementsByTagName：接受一个参数，元素的标签名，返回的是NodeList集合，该集合有namedItem方法，可以通过元素的name获取到元素

##### 文档写入

- write()：输出的任何HTML代码都能被处理，也可以动态引入外部资源，如js文件，</script>需要将/进行\转义
- writeln()：在write的字符串的末尾添加一个换行符(\n)
- open()：当write在页面加载后调用，会自动调用open方法
- close()：与open方法成对出现

#### Element
特征：

- nodeType的值为1
- nodeName的值为元素的标签名
- nodeValue的值为null
- parentNode可能是Document或Element

注：element.tagName表示输出标签名，但是在HTML中标签名始终以全部大写表示，而在XML中标签名与源代码保持一致
```allykeynamelanguage
element.tagName.toLowerCase() == "div"
```

##### HTML元素
所有的HTML元素都是由HTMLElement类型或者它的子类型表示

处理元素的特性：

- getAttribute：根据元素标签上特性的key值获取该key值对应的value，包括自定义特性，大小写不敏感，值总是字符串形式。通常只有在取自定义特征的情况下，才会使用getAttribute方法
    - style：通过getAttribute访问，返回的是css文本；通过属性访问，返回的是一个对象
    - onclick：通过getAttribute访问，返回的是相应代码的字符串；通过属性访问，返回集合js函数
- setAttribute：接受两个参数，特征名和值，该方法设置的特征名会统一转成小写形式
- removeAttribute：接受一个参数，特征名