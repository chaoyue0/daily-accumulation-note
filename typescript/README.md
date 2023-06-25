## TS

### 类型推导
````
const getType = (val: unknown) => Object.prototype.toString.call(val).slice(8, -1).toLowerCase()

const isArray = <T>(val: unknown): val is T[] => getType(val) === 'array'

````
#### is关键字
定义：类型谓语，用来判断一个变量属于某个接口或类型

场景：封装一个类型判断函数

### interface和type的异同

- 相同点：

1、都可以描述一个对象或函数

2、都允许拓展

- 区别：

1、interface拓展使用`extend`，type拓展使用`&`

2、type可以申明基本类型、联合类型、元祖类型

4、interface可以声明合并，相同接口名接口类型可以合并

3、type可以通过typeof获取实例的类型进行赋值
