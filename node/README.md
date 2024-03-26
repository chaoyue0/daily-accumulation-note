# node
原理：基于Chrome V8引擎构建的，由事件循环分发I/O任务，最终工作线程(Work Thread)将任务丢到线程池(Thread Pool)里去执行，
而事件循环只要等到执行结果即可

## 基础知识

### 函数传递
一个函数(`匿名函数`)可以作为变量在另一个函数的参数中进行传递
```javascript
function say(word) {
  console.log(word);
}

function execute(someFunction, value) {
  someFunction(value);
}

execute(say, "Hello");

//----------------------------

function execute(someFunction, value) {
    someFunction(value);
}

execute(function(word){ console.log(word) }, "Hello");
```

### 阻塞与非阻塞
在不新增额外线程的情况下，对任务进行并行处理，也就是Node是单线程的

#### exec
定义：创建子进程的函数，属于Node模块child_process

参数：command终端要运行的命令；回调函数作为参数，回调函数有三个参数分别是err、stdout、stderr

```javascript
export function useExec(shell: string) {
  return new Promise((res, rej) => {
    exec(shell, (err, stdout, stderr) => {
      if (err) rej(err);
      res({
        stdout,
        stderr,
      });
    });
  });
}
```

### 单线程
只有一个主线程，大大减少了线程间切换的开销，但会有多个worker线程，用于执行异步操作

#### 好处

- 占用内存小
- 不用切换线程降低CPU开销
- 线程安全

#### 劣势

- 处理CPU密集型任务，占用CPU时间长
- 无法利用CPU多核
- 抛出异常导致程序停止

#### 子进程
对于CPU密集型操作，可以通过child_process创建独立子进程，父子进程通过IPC通信

#### 线程与进程
##### 根本区别

- 线程：运算调度的最小单元
- 进程：资源分配的最小单元

##### 从属关系

- 进程：包含线程，一个进程可以有很多线程，每条线程并行执行不同的任务
- 线程：单一顺序的控制流
##### 开销

- 进程的创建、销毁和切换的开销都远大于线程

##### 拥有的资源

- 进程：拥有自己的内存和资源，进程中的线程会共享这些内存和资源；拥有一个完整的虚拟地址空间，同一进程内的不同线程共享同一地址空间

### 运行原理

- 应用层：js交互层，常见的是Node.js模块，如http、fs等
- V8引擎层：用来解析js语法，进而与下层API交互
- Node API层：为上层模块提供系统调用，与操作系统进行交互
- LIBUV层：跨平台底层封装，实现了事件循环、文件操作、线程池等，是Node实现异步的核心

Node.js是基于V8引擎构建的，由事件循环分发I/O任务，最终工作线程将任务丢到线程池里去执行，而事件循环只要等待执行结果即可

## Buffer
### 定义
缓冲区，用来处理二进制流数据或者与之进行交互，对于流式数据会采用缓冲区将数据临时存储起来，等缓冲区到一定的大小后存入硬盘中

    1个中文对应3位buffer字节

### 创建

- Buffer.from：从字符串、数组创建一个buffer
- Buffer.alloc：创建一个指定大小的buffer

### 内存机制
创建时大小已经被确定且无法调整，不再由V8分配，由C++层面完成申请，在JavaScript中进行内存分配，这部分内存称为`堆外内存`

### 内存分配原理
采用slab机制进行预先申请、事后分配的一种动态的管理机制

#### slab状态

- full：完全分配状态
- partial：部分分配状态
- empty：没有被分配状态

#### 8KB限制
Node以8kb为界限来区分是小对象还是大对象

## 工作方式

### 基于事件驱动的回调
创建了服务器，并且向创建它的方法传递了一个函数。无论何时我们的服务器收到一个请求，这个函数就会被调用

## Express 
### Middleware 中间件
定义：将具体的业务逻辑和底层逻辑解耦的组件，即中间件是能够适用多个应用场景、可复用性良好的代码

参数：request(http请求)、response(http响应)、next(代表下一个中间件)

特点：每个中间件都可以对http请求进行加工，并且决定是否调用next方法，将request对象传给下一个中间件

    next如果带有参数，则表示抛出一个错误，参数为错误文本，抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止

- 中间件是按照顺序执行的，配置中间件的顺序非常的重要
- 中间件在执行内部逻辑的时候可以选择将请求传递给下一个中间件，也可以直接返回用户响应

#### 原理

- 为了用`串行化流程`让几个异步任务按顺序执行，需要先把这些任务按预期的执行顺序放到一个数组中(起到`队列`的作用)
- 每个任务都是一个函数，任务完成后调用处理器函数，告知错误状态和结果(`next函数`)

```javascript
function next(err, result){
    if(err) throw err;
    var currentTask = tasks.shift();
    if(currentTask) currentTask(result)
    next();
}
```

    异步串行方案还可以使用promise的then链、async\await、yeild等

### express方法
#### use
定义：是express注册中间件的方法，返回一个函数

    use方法也允许将请求网址写在第一个参数，表示只有请求路径匹配这个参数，后面的中间件才会生效

#### http动词
get、post、put、delete方法

第一个参数都是请求的路径，第二个参数包含请求和响应的回调函数

#### set
用于指定变量的值，两个参数分别为key和value

#### response
##### redirect
允许网址的重定向
```javascript
response.redirect("http://www.example.com");
```

##### sendFile
用于发送文件
```javascript
response.sendFile("/path/to/anime.mp4");
```

##### render
用于渲染网页模板

#### request
##### ip
用于获取http请求的ip地址

##### files
用于获取上传的文件

##### http头信息
要指定http头信息，回调函数就必须换一种写法，要使用setHeader方法和end方法
```javascript
app.get('/', function(req, res){
  var body = 'Hello World';
  res.setHeader('Content-Type', 'text/plain');
  res.setHeader('Content-Length', body.length);
  res.end(body);
});
```

### path模块
是node用于处理文件\目录路径的内置模块，node中所有的模块都需要require在文件顶部导入

```javascript
const path = require("path");
```
#### basename 获取路径基础名
path.basename(path, ?ext):path路径的最后一部分

- path和ext必须为字符串
- ext和文件名后缀匹配上返回值会省略文件后缀
- path尾部的目录分隔符会被忽略

#### dirname 获取路径目录名
path.dirname(path):path路径的目录名

- path必须为字符串
- path尾部的目录分隔符会被忽略

#### extname 获取路径扩展名
path.extname(path)

#### parse 解析路径
path.parse(path)：带有(dir根目录,root文件所在文件夹,base完整文件,name文件名,ext后缀名)的对象

#### join 拼接路径
path.join([...paths]):平台特定的分隔符作为定界符将所有path片段连接在一起生成的路径

#### normalize 规范化路径
path.normalize(path):规范后的路径字符串

#### resolve 解析为绝对路径
从右往左的顺序一次解析，直到构造出一个绝对路径，否则会将当前工作目录加在路径开头

#### 全局变量

- __dirname：表示当前执行文件所在目录的`完整目录名`
- __filename：表示当前执行文件的`完整文件名`

### 配置路由
路由器功能成了一个单独的组件`Express.Router`

#### 基础用法
Express.Route是一个构造函数，调用后返回一个路由器实例
```javascript
const router = express.Router()
```

### 模板引擎
每个页面模板使用特殊的语法和变量来定义一个网页的基本结构

#### Pug
提供了一种基于缩进的语法来向页面添加内容，可以在代码中使用多种控制结构，包括条件语句和循环

- 直接使用html元素名，不包含标签
- 使用缩进推导层次结构
- 用前面的点来给一个元素声明css类

##### 访问变量

- 直接使用=分配给一个HTML元素或属性的唯一内容
- 使用#{var}来访问变量值(函数也行)

##### 在数组上循环操作
each item in arr

```
each field in fields 
    li.nav-item  
         a.nav-link(href="/"+field.name) #{field.name}
```

##### 条件判断
```
if name
  h2 Hello from #{name}
else
  h2 Hello
```

##### Express中使用Pug模板

- 安装Pug
- 设置视图引擎和专用的视图目录
- 请求渲染pug文件

```javascript
//设置视图引擎和专用的视图目录
app.set("view engine", "pug");
app.set("views", path.join(__dirname, "views"));
```

```javascript
//请求渲染pug文件
app.get("/", function(req, res) {
   res.render('index', {port: port});
});
```

#### Handlebars

- 无逻辑的模板引擎，不支持模板代码中基于逻辑的条件或循环
- 使用常规HTML语法来构建页面结构，而不依赖缩进

##### 访问变量
使用双大括号把变量括起来

##### 迭代
```
{{#each fields}}
    <li class="nav-item">
        <a class="nav-link" href="/{{name}}">{{name}}</a>
     </li>
{{/each}}
```

## config-lite
轻量的读取配置文件的模块

## mongoose
nodejs提供连接mongodb的库，对node环境中MongoDB数据库操作的封装，一个对象模型工具