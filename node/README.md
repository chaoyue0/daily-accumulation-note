# node

## 原理
基于 Chrome V8 引擎构建的，由事件循环分发 I/O 任务，最终工作线程(Work Thread)将任务丢到线程池(Thread Pool)里去执行，而事件循环只要等到执行结果即可

## 核心概念
- Chrome V8：解析并执行js代码
- libuv：由事件循环和线程池组成，负责所有 I/O 任务的分发与执行

## 基础知识

### 与浏览器的区别

- 没有 document 和 window 浏览器提供的所有对象
- 可以控制环境，无需转码兼容浏览器
- 同时支持 CommonJS 和 ES 模块系统

### 函数传递

一个函数(`匿名函数`)可以作为变量在另一个函数的参数中进行传递

```javascript
function say(word) {
  console.log(word)
}

function execute(someFunction, value) {
  someFunction(value)
}

execute(say, 'Hello')

//----------------------------

function execute(someFunction, value) {
  someFunction(value)
}

execute(function (word) {
  console.log(word)
}, 'Hello')
```

### 阻塞与非阻塞

在不新增额外线程的情况下，对任务进行并行处理，也就是 Node 是单线程的

#### exec

定义：创建子进程的函数，属于 Node 模块 child_process

参数：command 终端要运行的命令；回调函数作为参数，回调函数有三个参数分别是 err、stdout、stderr

```javascript
export function useExec(shell: string) {
  return new Promise((res, rej) => {
    exec(shell, (err, stdout, stderr) => {
      if (err) rej(err)
      res({
        stdout,
        stderr,
      })
    })
  })
}
```

### 单线程

只有一个主线程，大大减少了线程间切换的开销，但会有多个 worker 线程，用于执行异步操作

#### 好处

- 占用内存小
- 不用切换线程降低 CPU 开销
- 线程安全

#### 劣势

- 处理 CPU 密集型任务，占用 CPU 时间长
- 无法利用 CPU 多核
- 抛出异常导致程序停止

#### 子进程

对于 CPU 密集型操作，可以通过 child_process 创建独立子进程，父子进程通过 IPC 通信

#### IPC 进程间通信

每个进程有独立的地址空间，IPC 的目的是为了进程之间的资源可以共享访问，采用管道技术

##### 管道

自身也属于一个进程，用于连接两个进程，将一个进程的输出作为另一个进程的输入

##### Libuv

将 IO 操作交给 Libuv，保证主线程可以继续处理其他事情，为上层的 Node 提供了统一的 API 调用，无需考虑平台差异

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

- 应用层：js 交互层，常见的是 Node.js 模块，如 http、fs 等
- V8 引擎层：用来解析 js 语法，进而与下层 API 交互
- Node API 层：为上层模块提供系统调用，与操作系统进行交互
- LIBUV 层：跨平台底层封装，实现了事件循环、文件操作、线程池等，是 Node 实现异步的核心

Node.js 是基于 V8 引擎构建的，由事件循环分发 I/O 任务，最终工作线程将任务丢到线程池里去执行，而事件循环只要等待执行结果即可

### Buffer

#### 定义

缓冲区，用来处理二进制流数据或者与之进行交互，对于流式数据会采用缓冲区将数据临时存储起来，等缓冲区到一定的大小后存入硬盘中

    1个中文对应3位buffer字节

#### 创建

- Buffer.from：从字符串、数组创建一个 buffer
- Buffer.alloc：创建一个指定大小的 buffer

#### 内存机制

创建时大小已经被确定且无法调整，不再由 V8 分配，由 C++层面完成申请，在 JavaScript 中进行内存分配，这部分内存称为`堆外内存`

#### 内存分配原理

采用 slab 机制进行预先申请、事后分配的一种动态的管理机制

##### slab 状态

- full：完全分配状态
- partial：部分分配状态
- empty：没有被分配状态

##### 8KB 限制

Node 以 8kb 为界限来区分是小对象还是大对象

## Express 框架

### Middleware 中间件

定义：处理http请求的函数，可以访问请求对象、响应对象和下一个中间件函数，按照定义的顺序依次执行，核心作用是通过链式调用对请求进行分层处理

参数：request(http 请求)、response(http 响应)、next(代表下一个中间件)

- next如果带有参数，则表示抛出一个错误，参数为错误文本，抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止
- 中间件是按照顺序执行的，配置中间件的顺序非常的重要
- 中间件在执行内部逻辑的时候可以选择将请求传递给下一个中间件，也可以直接返回用户响应

#### 原理

- 为了用`串行化流程`让几个异步任务按顺序执行，需要先把这些任务按预期的执行顺序放到一个数组中(起到`队列`的作用)
- 每个任务都是一个函数，任务完成后调用处理器函数，告知错误状态和结果(`next函数`)

#### 分类
- 应用级中间件：绑定到整个应用，app.use()
- 路由级中间件：绑定特定路由，router.use()
- 错误处理中间件：捕获并处理错误
- 内置中间件：解析json格式的请求体，express.json()

#### 使用
- app.use(req,res,next)，单独写处理函数
- app.get('/', funArrays, (req, res)，多个处理函数写在funArrays中

### path 模块

是 node 用于处理文件\目录路径的内置模块，node 中所有的模块都需要 require 在文件顶部导入

```javascript
const path = require('path')
```

#### basename 获取路径基础名

path.basename(path, ?ext):path 路径的最后一部分

- path 和 ext 必须为字符串
- ext 和文件名后缀匹配上返回值会省略文件后缀
- path 尾部的目录分隔符会被忽略

#### dirname 获取路径目录名

path.dirname(path):path 路径的目录名

- path 必须为字符串
- path 尾部的目录分隔符会被忽略

#### extname 获取路径扩展名

path.extname(path)

#### parse 解析路径

path.parse(path)：带有(dir 根目录,root 文件所在文件夹,base 完整文件,name 文件名,ext 后缀名)的对象

#### join 拼接路径

path.join([...paths]):平台特定的分隔符作为定界符将所有 path 片段连接在一起生成的路径

#### normalize 规范化路径

path.normalize(path):规范后的路径字符串

#### resolve 解析为绝对路径

从右往左的顺序一次解析，直到构造出一个绝对路径，否则会将当前工作目录加在路径开头

#### 全局变量

- \_\_dirname：表示当前执行文件所在目录的`完整目录名`
- \_\_filename：表示当前执行文件的`完整文件名`

### 配置路由

路由器功能成了一个单独的组件`Express.Router`

#### 基础用法

Express.Route 是一个构造函数，调用后返回一个路由器实例

```javascript
const router = express.Router()
```

### 模板引擎

每个页面模板使用特殊的语法和变量来定义一个网页的基本结构

#### Pug

提供了一种基于缩进的语法来向页面添加内容，可以在代码中使用多种控制结构，包括条件语句和循环

- 直接使用 html 元素名，不包含标签
- 使用缩进推导层次结构
- 用前面的点来给一个元素声明 css 类

##### 访问变量

- 直接使用=分配给一个 HTML 元素或属性的唯一内容
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

##### Express 中使用 Pug 模板

- 安装 Pug
- 设置视图引擎和专用的视图目录
- 请求渲染 pug 文件

```javascript
//设置视图引擎和专用的视图目录
app.set('view engine', 'pug')
app.set('views', path.join(__dirname, 'views'))
```

```javascript
//请求渲染pug文件
app.get('/', function (req, res) {
  res.render('index', { port: port })
})
```

#### Handlebars

- 无逻辑的模板引擎，不支持模板代码中基于逻辑的条件或循环
- 使用常规 HTML 语法来构建页面结构，而不依赖缩进

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

### API
#### express
- express.json：解析传入请求为json
- express.raw：解析传入的请求为buffer
- express.text：解析传入的请求为字符串
- express.router；创建路由对象，可以将中间件和http方法添加到路由
- express.static：提供静态文件，包括图像、css、js文件
、、

#### app
##### 模块化
- app.on('mount', (parent) => {})：获取父应用的配置项和挂载路径

##### http
- app.get
- app.post
- app.all：匹配所有http动词，仅对精确路由生效，不会子路径，中间件的全局处理，如日志记录或验证

##### 路由
- app.param：处理路由参数，验证参数合法性
- app.path：返回应用的规范路径
- app.route：

##### 视图
- app.render(view, locals, callback)：配合模板文件渲染html，locals为传递给模板的变量数据对象

## 数据库
### 事务
#### 事务特性 ACID
- 原子性：作为一个整体执行，要么全部执行，要么全部不执行
- 一致性：确保从一个状态变成另一个状态
- 隔离性：一个事务的执行不会影响其他事务
- 持久性：事务一旦提交对数据库的修改就会永久保存在数据库中

#### 事务分类
- 隐式事务：单条sql语句
- 显式事务：讲多条sql语句作为一个事务执行，使用BEGIN开启一个事务，使用COMMIT提交一个事务，使用ROLLBACK回滚事务


#### 事务隔离级别
两个并行执行的事务，如果涉及到操作同一条记录，可能会带来数据的不一致

- read uncommitted 读未提交：一个事务可以读取另一个未提交事务的数据，可能会导致**脏读**
- read committed 读提交：一个事务要等到另一个事务提交后才能读取数据，两个相同的查询返回不同的数据，可能会导致**不可重复读**（对应的是修改操作）
- repeatable read 重复读：开始读取数据时，不再允许修改操作，可能会导致**幻读**（对应的是插入操作）
- serializable 序列化：串行化顺序执行，级别最高，但效率低下，耗费数据库性能 


### 索引优化
索引是数据库的目录，但是索引会占用额外的存储空间

#### 索引类型
- B+树索引：支持范围查询和排序
- 哈希索引：仅适用于等值查询
- 全文索引：针对文本的模糊匹配
- 位图索引：适合低基数数列，如性别
- 覆盖索引：索引包含查询所需所有字段

#### 索引设计原则
- 选择性原则：优先为区分度高（该列不同值多，如id无重复值）的列建索引
- 最左前缀匹配：复合索引按最左列优先匹配查询条件
- 避免冗余索引：不重复创建功能相同的索引，已有索引 (A, B)，再建 (A) 是冗余的，但 (B) 或 (B, A) 不冗余
- 短索引优化：对于长字段使用前缀索引，减少空间占用
- 覆盖索引：索引包含查询所需的所有字段，避免回表查询
- 热点数据分离：频繁更新的字段谨慎建索引
