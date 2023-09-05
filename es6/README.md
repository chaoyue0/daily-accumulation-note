# ES6
## 参数列表
在ES6之前采用的是`arguments对象`来获取参数列表，存在一些限制和不便

- 不支持数据方法：arguments对象只是类数组对象，没有forEach、map、length方法和属性
- 无法使用箭头函数：箭头函数没有自己的arguments对象，会继承外层的arguments对象

ES6新增了剩余参数列表语法，`...args`，允许函数接收可变数量的参数，并将这些参数放入一个数组中。
可以和其他参数一同使用，但是必须是参数列表的最后一个参数

## 强引用和弱引用
弱引用和强引用相对，是指弱引用其引用的对象会自动被垃圾回收机制回收

### Map和WeakMap

- Map的键可以是任何类型，WeakMap的键只接受对象类型(null除外)
- Map的键是跟内存地址绑定的，只要内存地址不同就看成两个键；而WeakMap的键是弱引用，键所指的对象可以被垃圾回收，此时键是无效的
- Map可以被遍历；WeakMap不能被遍历

## 解构

- 数组解构
```typescript
// ...解构
let [head, ...tail] = [1, 2, 3, 4]
console.log(head, tail) // 1, [2, 3, 4]

// 解构不成功为undefined
let [a, b, c] = [1]
console.log(a, b, c) // 1, undefined, undefined

// 解构默认赋值
let [a1 = 1, b1 = 2] = [3]
console.log(a1, b1) // 3, 2
```
- 对象解构
```typescript
// 可以不按照顺序，这是数组解构和对象解构的区别之一
let { f2, f1 } = { f1: 'test1', f2: 'test2' }
console.log(f1, f2) // test1, test2

// 解构对象重命名
let { f3: rename, f4 } = { f1: 'test1', f2: 'test2' }
console.log(rename, f4) // test1, test2
```
- String、Map、Set
```typescript
// String
let [ a, b, c, ...rest ] = 'test123'
console.log(a, b, c, rest) // t, e, s, [ 't', '1', '2', '3' ]

// Map
let [a1, b1] = new Map().set('f1', 'test1').set('f2', 'test2')
console.log(a1, b1) // [ 'f1', 'test1' ], [ 'f2', 'test2' ]

// Set
let [a2, b2] = new Set([1, 2, 3])
console.log(a2, b2) // 1, 2
```

### 解构原理
`可迭代对象`和`Iterator接口`组成的语法糖，通过遍历器按顺序获取对应的值进行赋值

##### Iterator 接口
为各种不一样的数据结构提供统一的访问机制，任何数据结构只要有Iterator接口，就能通过遍历操作，依次按顺序处理数据结构中的所有成员，主要为`for of`服务

    在js中没有接口的概念，可以将Iterator理解成一种特殊的对象，即迭代器对象，返回的方法叫做迭代器方法

迭代器方法`next()`，返回值是一个`object`，包含两个属性，`value`和`done`，其中value表示具体的返回值，done是布尔类型表示集合是否遍历完成或后续是否还有可用数据
```typescript
function getIterator(list) {
    var index = 0;
    return {
        next: function() {
            var done = (index >= list.length);
            var value = !done ? list[index++] : undefined;
            return {
                done: done,
                value: value
            };
        }
    };
}
```

##### 可迭代对象
定义：指的是一种协议，任何遵循该协议的对象都能成为可迭代对象

- 可迭代协议：对象或原型链上必须有一个名叫`Symbol.iterator`的属性，值是无参函数，该函数返回一个对象，该对象符合迭代器协议
- 迭代器协议：定义一个函数来产生一个有限或无限序列值，类似上述`getIterator函数`，包含next方法，该方法返回done和value属性

String、Array、Map、Set等原生数据结构都是可迭代对象

    原生object对象是默认没有部署Iterator接口，即object不是一个可迭代对象，因此不能使用for of遍历object
    原因：对象的哪一个属性先后遍历是不确定的，需要开发者手动指定。而本质上，迭代器是一种线性处理，而对象的处理属于非线性处理

## promise
定义：promise是一个规范，实现promise的方法有很多，但是必须满足标准定义，也就是`promise/A+规范`

### promise/A+ 规范
#### promise state
一个promise必须包含3个状态：pending、fulfilled、rejected

#### promise method
一个promise必须提供一个`then方法`，该方法接受两个参数：`onFulfilled`和`onRejected`，且必须返回一个promise



### 构造函数
#### 第一阶段
```typescript
function Promise(executor) {
  var self = this
  self.status = 'pending' // Promise当前的状态
  self.data = undefined  // Promise的值
  self.onResolvedCallback = [] // Promise resolve时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = [] // Promise reject时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面

  executor(resolve, reject) // 执行executor并传入相应的参数
}
```
    1、executor函数执行过程中可能会报错
    2、executor函数的两个参数resolve、reject未定义

#### 第二阶段
针对executor函数执行过程中可能会报错的情况
```typescript
  try { // 考虑到执行executor的过程中有可能出错，所以我们用try/catch块给包起来，并且在出错后以catch到的值reject掉这个Promise
    executor(resolve, reject) // 执行executor
  } catch(e) {
    reject(e)
  }
```

#### 第三阶段
针对executor函数两个未定义的参数
```typescript
 function resolve(value) {
    if (self.status === 'pending') {
      self.status = 'resolved'
      self.data = value
      for(var i = 0; i < self.onResolvedCallback.length; i++) {
        self.onResolvedCallback[i](value)
      }
    }
  }

  function reject(reason) {
    if (self.status === 'pending') {
      self.status = 'rejected'
      self.data = reason
      for(var i = 0; i < self.onRejectedCallback.length; i++) {
        self.onRejectedCallback[i](reason)
      }
    }
  }
```
判断status为pending时，将status改成对应的值(resolved / rejected)，并将value和reason存到data上，之后执行相应的回调函数

### then方法
用来注册这个Promise状态确认后的回调，需要将then方法写在原型链上，并返回一个新的Promise对象

#### 链式调用
每个promise对象都可以多次调用then方法，每次调用返回的promise状态取决于那一次调用时传入参数的返回值

#### 源码实现



## arrow function
### 语法
```typescript
// 当只有一个参数时，圆括号是可选的：
singleParam => { statements }

// 没有参数的函数应该写成一对圆括号：
() => { statements }

//加括号的函数体返回对象字面量表达式：
params => ({foo: bar})

//同样支持参数列表解构
let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c;
f();  // 6
```

### 作用

- 简短的函数写法
- 不绑定this，没有单独的this


    在箭头函数出现之前，每一个函数根据它是被如何调用的来定义这个函数的this值
        - 如果函数是构造函数，this指针指向实例对象
        - 如果函数是一个对象的方法，this指针指向这个对象

### 函数体
箭头函数可以有一个`简写体`和常见的`块体`

在简写体中，只需要一个表达式，并附加一个隐式的返回值
```typescript
var func = (x) => x * x;
```
在块体中，必须明确使用return语句
```typescript
var func = (x, y) => {
  return x + y;
};
```

### 注意项

- 箭头函数`不能使用new操作符`，和new一起用会抛出错误
- 箭头函数`没有prototype属性`，返回undefined

## class
定义：es6中新加入的继承机制，实际是js关于`原型继承机制`的语法糖，本质上是对原型继承的封装