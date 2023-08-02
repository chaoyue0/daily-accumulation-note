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
- createElement：接受一个参数，要创建元素的标签名



#### Text

- nodeType值为3
- nodeValue值为节点所包含的文本
- parentNode是一个Element
- 没有子节点

处理文本节点

- appendData(text)：将text添加到节点的末尾
- deleteData(offset,count)：从offset指定位置开始删除count个字符
- insertData(offset,text)：在offset指定的位置插入text
- replaceData(offset,count,text)：用text替换在offset位置开始到offset+count位置的元素
- splitText(offset)：从offset指定的位置将当前文本节点分成两个文本节点
- substringData(offset,count)：提取offset位置开始到offset+count位置的元素

#### Attr

- nodeType的值为2
- nodeName的值为特征的名称
- nodeValue的值为特征的值
- parentNode的值为null
- 没有子节点

处理属性节点

- getAttribute(key)
- setAttribute(key,value)
- removeAttribute(key)

#### DOM操作技术
##### 动态脚本
定义：值的是页面加载时不存在，但将来的某个时刻通过修改DOM动态添加的脚本

创建方式：

- 插入外部文件
- 直接插入JS代码

##### 动态样式
定义：将css样式包含到html页面中

创建方式：

- link引入式css
- style嵌入式css

注：加载外部样式文件的过程是异步的，与js代码的执行没有固定的次序

##### 操作表格

table子标签：

- caption：常作为table标签的第一个子元素出现，表示表格的标题
- thead：定义一组表格行的列头
- tbody：封装了一系列表格的行，作为表格的主体
- tfoot：定义一组表格列的汇总行

table属性：

- rows：一个表格中所有行的HTMLCollection

table方法：

- createXXX：XXX为子标签名，表示创建元素
- deleteXXX：XXX为子标签名，表示删除元素
- deleteRow(x)：删除指定位置的行
- insertRow(x)：在rows集合中指定位置插入一行

##### NodeList
NodeList、NameNodeMap、HTMLCollection三者都是动态的集合，每当文档发生变化，它们都会得到更新

涉及DOM对性能的影响，NodeList这类动态的对象，意味着每次访问NodeList对象，都会运行一次查询，因此尽可能减少DOM操作

    所有NodeList对象都是在访问DOM文档时实时运行的查询

### DOM扩展
主要的扩展内容是`选择符API`和`HTML5`

#### 选择符API

- querySelector：接受一个参数，css选择器，返回与该模式匹配的第一个元素，否则返回null
- querySelectorAll：返回NodeList的实例，否则返回空，底层实现类似一组元素的快照，而非不断对文档进行搜索的动态查询

要取得NodeList中的每一个元素，可以使用item()方法，也可以使用方括号下标语法
```allykeynamelanguage
for (i=0, len=strongs.length; i < len; i++){
    strong = strongs[i]; //或者 strongs.item(i)
}
```

- matchesSelector / matches：接受一个参数，css选择器，如果元素被指定的选择器选择则返回true，否则返回false
```allykeynamelanguage
let result = element.matches(selectorString);
```

#### 元素遍历
新增属性：

- childElementCount：返回子元素的个数，不包含文本节点和注释
- firstElementChild：指向第一个子元素
- lastElementChild：指向最后一个子元素
- previousElementSibling：指向前一个兄弟元素
- nextElementSibling：指向后一个兄弟元素

#### HTML5
##### 与类相关的补充

- getElementsByClassName方法

接受一个参数，即包含一或多个类名的字符串，返回NodeList，和getElementsByTagName以及其他返回NodeList的DOM方法一样具有同样的性能问题

- classList属性

可以通过className属性进行添加、删除和替换类名，由于className是一个字符串，即使只修改部分字符串，也必须设置整个字符串的值。
classList属性是新集合类型`DOMTokenList实例`，有length属性，要取得每个元素可以使用item方法或方括号语法，

新的方法：

1、add：将给定的字符串值添加到列表中，若值已经存在就不添加

2、contains：判断列表中是否存在给定的值，存在返回true，否则返回false

3、remove：从列表中删除给定的字符串

4、toggle：如果列表中存在给定值删除它，如果不存在给定值添加他



##### 焦点管理
元素获取焦点的方式：

- 页面加载
- 用户输入(Tab键)
- 在代码中使用focus()

引用DOM中当前获取了焦点的元素：document.activeElement

    默认情况下，文档刚刚加载完成时，document.activeElement返回的是document.body元素，文档加载期间，document.activeElement返回的是null

判断文档是否获得了焦点：document.hasFocus

    通过检查文档是否获得了焦点，可以知道用户与页面的交互情况



##### HTMLDocument的变化

- readyState属性：表示文档是否加载完成，两个属性值loading表示正在加载文档，complete表示已经加载完文档
- compatMode属性：表示浏览器采用的渲染模式，两个属性值CSS1Compat表示标准模式，BackCompat表示混杂模式
- head属性：引用文档的head元素

##### 字符集属性

- charset属性：表示文档中实际使用的字符集，可以直接通过charset属性修改这个值

##### 自定义数据属性
HTML5规定可以为元素添加非标准属性，但必须添加前缀`data-`，目的是为元素提供与渲染无关的信息，或提供语义信息

dataset属性：可以访问自定义数据属性，是一组键值对的映射形式

##### 插入标记

- innerHTML属性：在读模式下，返回从对象的起始位置到终止位置的全部内容,不包括标签；在写模式下，该属性会根据指定的值创建新的DOM树

限制：
1、通过innerHTMl插入script元素并不会执行其中的脚本

2、不支持innerHTML的元素：col、head、html、style、table、title

- outerHTML属性：在读模式下，该属性会返回调用它的元素及所有子节点的HTML标签，包括标签本身；在写模式下，该属性会创建新的DOM树，用新DOM树替换原先的所有子节点
- insertAdjacentHTML()方法：接受两个参数，插入位置和要插入的HTML文本

插入位置：

1、beforebegin：在当前元素之前插入一个紧邻的兄弟节点

2、afterbegin：在当前元素之下插入一个新的子元素或在第一个子元素之前插入一个新的子元素

3、beforeend：在当前元素之下插入一个新的子元素或在最后一个子元素之后插入一个新的子元素

4、afterend：在当前元素之后插入一个紧邻的兄弟节点

##### 页面滚动

scrollIntoView()方法：通过滚动浏览器窗口或某个容器元素，调用元素出现在视口中。

接受两个参数，均可选：

- alignToTop：布尔值，如果true表示元素的顶端将和它所在的滚动区域的顶端对齐，如果是false表示元素的底端与所在滚动区域的底端对齐
- scrollIntoViewOptions：对象，behavior属性定义动画过度效果auto或smooth，block属性定义垂直方向的对齐，inline属性定义水平方向的对齐

#### 专有扩展
##### children属性
定义：是HTMLCollection的实例，包含元素中同样还是元素的子节点

##### contains方法
定义：用来判断某个节点是不是另一个节点的后代，不需要通过在DOM文档树中查找。调用该方法的是祖先节点，接受的参数是要检测的节点

##### 插入文本

- innerText：操作元素中包含的`所有`文本内容，包含子文档书中的文本。对于HTML语法字符(&、<b>)会进行编码
  - 读取值：按照由浅入深的顺序，将子文档树中的所有文本拼接起来
  - 写入值：删除元素的所有子节点，插入包含相应文本值的文本节点

设置innerText永远只会生成当前节点的一个子文本节点，为了确保只生成一个子文本节点，必须对文本进行HTML编码

通过innerText属性过滤吊HTML标签
```allykeynamelanguage
div.innerText = div.innerText;
```

- outerText：除了作用范围扩大到了包含调用它的节点之外，outerText和innerText基本上没有区别

##### 滚动
HTMLElement类型的拓展，所有元素中都可以调用

- scrollIntoViewIfNeeded：只在当前元素在视口中不可见的情况下，才滚动浏览器窗口，最终让它可见。如果参数设置为true，则表示尽可能让元素在视口垂直居中
- scrollByLines：参数可以是正值，也可以是负值，比如1表示页面主题滚动1行，表示将元素的内容滚动指定的行高，`其中一行的高度通常由元素内文本的字体大小和行高(line-height)来决定`
- scrollByPages：参数可以是正值，也可以是负值，比如1表示页面主题滚动1页，表示当前文档页面按照指定的页数翻页

注意点：
    
    scrollIntoViewIfNeeded的作用对象是元素的容器，scrollByLines和scrollByPages影响的是元素本身
    scrollIntoViewIfNeeded、scrollByLines、scrollByPages在Safari和Chromo浏览器中生效

### DOM2和DOM3
判断浏览器是否支持DOM模块
```allykeynamelanguage
document.implementation.hasFeature("Core", "2.0");
```
#### DOM变化
##### XML命名空间
有了XML命名空间，不同XML文档的元素可以混在一起使用，不用担心命名冲突，命名空间使用xmlns特性来指定

```allykeynamelanguage
<xhtml:html xmlns:xhtml="http://www.w3.org/1999/xhtml">
        <xhtml:head>
            <xhtml:title>Example XHTML page</xhtml:title>
        </xhtml:head>
        <xhtml:body xhtml:class="home">
            Hello world!
        </xhtml:body>
</xhtml:html>
```

1、Node类型的变化

与命名空间相关的属性(DOM2)：

- localName：不带命名空间前缀的节点名称
- namespaceURI：命名空间URI(表示元素或属性所在的命名空间的唯一标识符)，未指定的话返回null
- prefix：命名空间前缀，未指定的话返回null

与命名空间相关的方法(DOM3)：

- isDefaultNamespace(namespaceURI)：在指定的 namespaceURI 是当前节点的默认命名空间的情况下返回 true
- lookupNamespaceURI(prefix)：返回给定 prefix 的命名空间
- lookupPrefix(namespaceURI)：返回给定 namespaceURI 的前缀

2、Document类型的变化

与命名空间相关的方法(DOM2)：

- createElementNS(namespaceURI，tagName)：使用给定的tagName创建一个属于命名空间namespaceURI的新元素
- createAttributeNS(namespaceURI，attributeName)：使用给定的attributeName创建一个属于命名空间namespaceURI的新元素
- getElementsByTagNameNS(namespaceURI，tagName)：返回属于命名空间namespaceURI的tagName元素的NodeList

3、Element类型的变化

- getAttributeNS(namespaceURI,localName)：取得属于命名空间且名为localName的特性
- getAttributeNodeNS(namespaceURI,localName)：取得属于命名空间且名为localName的特性节点
- getElementsByTagNameNS(namespaceURI,tagName)：返回属于命名空间的tagName元素的NodeList
- hasAttributeNS(namespaceURI,localName)：确定当前元素是否有一个名为localName的特性
- removeAttributeNS(namespaceURI,localName)：删除属于命名空间且名为localName的特性
- setAttributeNS(namespaceURI,qualifiedName,value)：设置属于命名空间且名为qualifiedName的特性的值为value

除了第一个参数之外，其他的参数与DOM1中的方法相同

##### 其他方面的变化
1、Document类型的变化

- importNode方法：从一个文档中取得一个节点，然后将其导入到另一个文档，使其成为这个文档结构的一部分
- ownerDocument属性：表示节点所属的文档

2、Node类型的变化

- isSupported方法：表示确定当前节点是否具有什么能力，接受两个参数，特性名和特性版本号，返回布尔值
- isSameNode、isEqualNode方法：接受一个节点参数，在传入节点与引用节点`相同`或者`相等`时返回true

所谓相同指的是两个节点引用的是同一个对象

所谓相等指的是他们的attribute和childNodes属性也相等

- setUserData方法：表示将数据指定给节点，接受3个参数，要设置的键、实际的数据和处理函数

处理函数：表示带有数据的节点被复制、删除、重命名或引入一个文档时调用，
接受5个参数：表示操作类型的数值(1:复制、2:导入、3:删除、4:重命名)、数据键、数据值、源节点和目标节点，
删除节点时，源节点是null；除了复制节点时，目标节点均为null

#### 样式
##### 访问元素的样式
在css中属性使用短划线连接不同的词汇，如background-image，必须将其转换成驼峰大小写的形式，才能通过js来访问

    在标准模式下，所有度量值都必须指定一个度量单位；在混杂模式下，设置值就行浏览器会默认假设它是px单位
    document.compatMode，返回浏览器的渲染模式

为style对象(元素标签上的style特性)定义了一些属性和方法:

- cssText：表示css代码
- length：表示css属性的数量
- getPropertyValue(propertyName)：返回包含给定属性的字符串值
- item(index)：返回给定位置的css属性名称
- removeProperty(propertyName)：从样式中删除给定属性
- setProperty(propertyName,value,priority)：将给定属性设置为相应的值，并加上优先权标志(important)

    
    只能获取到元素的行内样式style

计算的样式：

getComputedStyle(element,null/伪元素字符串)：获取元素的样式，甚至可以`获取伪类元素样式`

    无论在哪个浏览器，所有计算的样式都是只读的，而且可以获取到所有的样式，包括行内样式和内嵌式样式

##### 操作样式表
CSSStyleSheet类型表示的是样式表，继承自StyleSheet，从其中接口继承了一系列属性

- disabled：表示样式表是否被禁止，属性是读写的
- href：如果样式表通过link标签包含，返回样式表的URL，否则返回null
- cssRules：样式表中包含的样式规则的集合
- deleteRule(index)：删除规则集合中指定位置的规则
- insertRule(rule,index)：向规则集合中指定位置插入rule字符串

1、CSSRule
```allykeynamelanguage
var sheet = document.styleSheets[0]; //返回样式集合
var rules = sheet.cssRules || sheet.rules; // 取得规则相关信息
var style = rules.style // 取得css样式表
```

- cssText：返回整条规则对应的文本
- selectorText：返回当前规则的选择符文本

```allykeynamelanguage
rule.style.backgroundColor = "red"
```
    通过这种方法修改规则会影响页面中适用于该规则的所有元素


##### 元素大小
1、偏移量

定义：包含元素在屏幕上占用的所有可见的空间，元素的可见大小由其高度、宽度决定，包括所有内边距、滚动条和边框大小(不包含外边框)

- offsetHeight：元素在垂直方向上占用的空间大小，包含元素的高度、内边距、`边框，以及(可见)水平滚动条的高度`
- offsetWidth：元素在水平方向上占用的空间大小，包含元素的高度、内边距、`边框，以及(可见)垂直滚动条的宽度`
- offsetLeft：元素的左边框至包含元素的左边框之间的像素距离
- offsetTop：元素的上边框至包含元素的上边框之间的像素距离

2、客户区大小

定义：元素内容及其内边距所占据的空间大小，包含高度、宽度以及内边距

- clientWidth：元素内容宽度 + 内边距宽度
- clientHeight：元素内容高度 + 内边距高度

3、滚动大小

定义：包含滚动内容的元素的大小，有些元素(HTML)会自动添加滚动条，有些元素需要添加css的overflow属性才能滚动

- scrollHeight：不通过滚动条隐藏内容的情况下，元素内容的总高度
- scrollWidth：不通过滚动条隐藏内容的情况下，元素内容的总宽度
- scrollLeft：被隐藏在区域左侧的像素数，通过设置这个属性可以改变元素的滚动位置
- scrollTop：被隐藏在区域上方的像素数，通过设置这个属性可以改变元素的滚动位置

4、确定元素大小

getBoundingClientRect方法，返回一个矩形对象(包含padding和border)，包含4个属性：left、top、right、bottom，所有的距离都是相对于视角窗口的左上角计算的

- left：左边界到元素左边界的距离
- right：左边界到元素右边距的距离
- top：上边界到元素上边界的距离
- bottom：上边界到元素下边界的距离

#### 遍历
##### NodeIterator
通过`document.createNodeIterator()`方法创建其实例，接收4个参数

- root：表示作为遍历根节点的节点
- whatToShow：表示该访问哪些节点，参数是一个位掩码
- filter：可以作为一个函数，表示是否接受或跳过特定节点
- entityReferenceExpansion：表示是否扩展实体引用，一般为false

实例方法

- nextNode()：表示在深度遍历中前进一步，遍历到最后一个节点时候，返回null
- previousNode()：表示在深度遍历中后退一步，遍历到根节点的时候，返回null

##### TreeWalker
通过`document.createTreeWalker()`方法创建其实例，与NodeIterator方法有相同的参数

与NodeIterator类似，同样拥有nextNode和previousNode方法，此外还添加了几个新的方法

- parentNode()：遍历到当前节点的父节点
- firstChild()：遍历到当前节点的第一个子节点
- lastChild()：遍历到当前节点的最后一个子节点