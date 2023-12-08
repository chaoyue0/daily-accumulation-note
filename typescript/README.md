# TS
## interface和type的异同

- 相同点：

1、都可以描述一个对象或函数

2、都允许拓展

- 区别：

1、interface拓展使用`extend`，type拓展使用`&`

2、type可以申明基本类型、联合类型、元祖类型
```typescript
// 基本类型别名
type Name = string

// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}

type Pet = Dog | Cat

// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]
```

4、interface可以声明合并，相同接口名接口类型可以合并
```allykeynamelanguage
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
```

5、type可以通过typeof获取实例的类型进行赋值

## 条件类型

    null和undefined是其他很合类型(包括void)的子类型，可以赋值给其他类型，赋值后都会变成null和undefined
infer可以在extends的条件语句中推断待推断的类型，infer表示一个占位

- infer只能在extends的右侧使用
- infer待推断的类型只能在三元表达式的true条件下
- infer只能搭配extends使用

### 推断函数的返回值类型
```typescript
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : T;

type func = () => number;
type variable = string;
type funcReturnType = MyReturnType<func>; // funcReturnType 类型为 number
type varReturnType = MyReturnType<variable>; // varReturnType 类型为 string
```

### 解包
#### 获取数组中的元素类型

```typescript
type Unpacked<T> = T extends (infer R)[] ? R : T;

Unpacked<number[]> //T:number[] => R:number Unpacked<T>:number
Unpacked<string> //T:string => T:string Unpacked<T>:string
```

1、 (infer R)[] 表示一个数组类型，infer R是用于推断数组中元素类型的类型变量

2、根据传入的类型T推断出数组中元素的类型

#### 递归来获取多层类型
```typescript
type Unpacked<T> = T extends Promise<infer R> ? Unpacked<R> : T;

type MyResponse = Promise<Promise<Promise<number>>>; //number
```

### 推断模板字符串
```typescript
type PickValue<T> = T extends `${infer R}%` ? R : unknown;

type Value = PickValue<"50%"> // "50"
```
##### 去除左侧空格
```typescript
type TrimLeft<T extends string> = T extends ` ${infer R}` ? TrimLeft<R> : T;

type Value = TrimLeft<"     value">; // "value"
```
` ${infer R}`模板字符串中在$前保留了一个空格，用于和T进行匹配，条件判断为true时，得到的值为去掉一个空格后的模板字符串，一直递归调用，如果不以空格开头则直接返回T

### 推断联合类型
```typescript
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;

type T11 = Foo<{ a: string; b: number }>; // T11类型为 string | number
```
{ a: string; b: number } 与 { a: infer U; b: infer U } 进行匹配，得到U的类型：string | number

#### 元组转联合类型
```typescript
type ElementOf<T> = T extends (infer R)[] ? R : never;

type TTuple = [string, number];
type Union = ElementOf<TTuple>; // Union 类型为 string | number
```

### 遍历数组
```typescript
type ReverseArray<T extends unknown[]> = T extends [infer First, ...infer Rest]  
? [...ReverseArray<Rest>, First]  
: T  
type Value = ReverseArray<[1, 2, 3, 4]> // [4,3,2,1]
```
[1, 2, 3, 4] 与 [infer First, ...infer Rest] 进行匹配，1对应First，234对应扩展运算符...Rest，匹配成功将1也就是First放到数组的尾部，递归调用直到Rest数组为空，递归调用停止

### 求类型的差集和交集
```typescript
type Diff<T, U> = T extends U ? never : T; // 找出T的差集
type Filter<T, U> = T extends U ? T : never; // 找出交集
```

## 索引类型
通过`索引签名`从对象中获取一些属性的值，在ts中索引签名可以是`string`、`number`或`symbols`

    在js中一个对象的索引签名会隐式调用toString方法，而在ts中会报错

场景：获取对象中一些属性的值，无法确定对象中是否包含某个属性
```typescript
function getValues<T, K extends keyof T>(person: T, keys: K[]): T[K][] {
  return keys.map(key => person[key]);
}
```
keyof 索引类型查询操作符：表示类型T所有公共属性的联合类型

T[k] 索引访问操作符：表示对象T的属性k

K extends T 泛型约束：限制k的属性只能是T对象属性的子集

## 声明空间
### 类型声明空间

- 在ts中用来存储类型声明的地方，包括`接口、类型别名、枚举`等类型声明
- 编译后不会存在于js中，只在ts编译阶段用来进行类型检查和类型推断
- 主要用于描述代码中的数据结构和类型信息
### 变量声明空间

- 在ts中用来存储变量和函数声明的地方，包括`变量、函数、类`等
- 编译后会生成相应的js代码，作为实际运行的实体
- 主要用于描述代码中的实际运行逻辑，包括存储数据和执行操作

## 模块
### 全局模块
在默认情况下，在一个新的ts文件下写的变量，它处于全局命名空间中，在其他ts文件中可以直接使用
```typescript
const foo = 123; // foo.ts
const bar = foo; // allowed bar.ts
``` 

### 文件模块
如果ts文件中含有import或export，那么它会在这个文件中创建一个本地的作用域，在其他ts文件中需要`显式的导入`
```typescript
import { foo } from './foo';
```

#### 模块路径

- 相对路径
- 动态查找

当导入路径不是相对路径时，模块解析将会模仿`Node模块解析策略`，会在node_modules里查找，会向`上级目录遍历`，直到系统的根目录，找到要加载的模块




#### 重写类型的动态查找
通过declare module 声明一个全局模块的方式
```typescript
declare module 'foo' {
  // some variable declarations
  export var bar: number;
}
```

## 命名空间
定义：命名空间是一种在ts和js中用于组织代码的方式，它可以跨多个文件使用

在A页面定义了一个命名空间，而在B页面要使用这个命名空间，需要在B页面引入A页面的命名空间

- 通过script标签加载A页面的脚本文件
- 通过ES模块import和export语法

## 类型
### 特殊类型
any、null、undefined、void

    null和undefined字面量和any类型的变量一样，都能被赋值给任意类型的变量

### 泛型
#### 类型推导
````typescript
const getType = (val: unknown) => Object.prototype.toString.call(val).slice(8, -1).toLowerCase()

const isArray = <T>(val: unknown): val is T[] => getType(val) === 'array'

````
##### is关键字
定义：类型谓语，用来判断一个变量属于某个接口或类型，通常与unknown搭配使用

场景：封装一个类型判断函数

### 联合类型
定义：使用` | `作为标记，如 string | number
```typescript
function formatCommandline(command: string[] | string) {
  let line = '';
  if (typeof command === 'string') {
    line = command.trim();
  } else {
    line = command.join(' ').trim();
  }
}
```

### 交叉类型
定义：使用` & `作为标记，可以从两个`对象`中创建一个新对象，新对象拥有两个对象所有的功能

    如果两个对象存在相同的属性，属性类型不同则返回never类型

### 元组 Tuple
定义：合并了`不同类型`的数组

- 可以通过`索引`对其中一部分元素访问或赋值，但是对元组类型的变量进行初始化时，需要提供所有元组类型中指定的项
- 元组类型赋值时，需要按照元组指定的类型`顺序`进行赋值
- 当添加越界元素时，该元素会被定义为元组中每个类型的联合类型

### 类型别名
使用`type`创建类型别名，常用于联合类型
```typescript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
```