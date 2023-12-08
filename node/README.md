# node

## Express Middleware 中间件
定义：将具体的业务逻辑和底层逻辑解耦的组件，即中间件是能够适用多个应用场景、可复用性良好的代码

参数：request(http请求)、response(http响应)、next(代表下一个中间件)

特点：每个中间件都可以对http请求进行加工，并且决定是否调用next方法，将request对象传给下一个中间件

    next如果带有参数，则表示抛出一个错误，参数为错误文本，抛出错误以后，后面的中间件将不再执行，直到发现一个错误处理函数为止

- 中间件是按照顺序执行的，配置中间件的顺序非常的重要
- 中间件在执行内部逻辑的时候可以选择将请求传递给下一个中间件，也可以直接返回用户响应

## express方法
### use
定义：是express注册中间件的方法，返回一个函数

    use方法也允许将请求网址写在第一个参数，表示只有请求路径匹配这个参数，后面的中间件才会生效

### http动词
get、post、put、delete方法

第一个参数都是请求的路径，第二个参数包含请求和响应的回调函数

### set
用于指定变量的值，两个参数分别为key和value

### response
#### redirect
允许网址的重定向
```javascript
response.redirect("http://www.example.com");
```

#### sendFile
用于发送文件
```javascript
response.sendFile("/path/to/anime.mp4");
```

#### render
用于渲染网页模板

### request
#### ip
用于获取http请求的ip地址

#### files
用于获取上传的文件

#### http头信息
要指定http头信息，回调函数就必须换一种写法，要使用setHeader方法和end方法
```javascript
app.get('/', function(req, res){
  var body = 'Hello World';
  res.setHeader('Content-Type', 'text/plain');
  res.setHeader('Content-Length', body.length);
  res.end(body);
});
```

## 配置路由

## 模板引擎
每个页面模板使用特殊的语法和变量来定义一个网页的基本结构

### Pug
提供了一种基于缩进的语法来向页面添加内容，可以在代码中使用多种控制结构，包括条件语句和循环

- 直接使用html元素名，不包含标签
- 使用缩进推导层次结构
- 用前面的点来给一个元素声明css类

#### 访问变量

- 直接使用=分配给一个HTML元素或属性的唯一内容
- 使用#{var}来访问变量值(函数也行)

#### 在数组上循环操作
each item in arr

```
each field in fields 
    li.nav-item  
         a.nav-link(href="/"+field.name) #{field.name}
```

#### 条件判断
```
if name
  h2 Hello from #{name}
else
  h2 Hello
```

#### Express中使用Pug模板

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

### Handlebars

- 无逻辑的模板引擎，不支持模板代码中基于逻辑的条件或循环
- 使用常规HTML语法来构建页面结构，而不依赖缩进

#### 访问变量
使用双大括号把变量括起来

#### 迭代
```
{{#each fields}}
    <li class="nav-item">
        <a class="nav-link" href="/{{name}}">{{name}}</a>
     </li>
{{/each}}
```