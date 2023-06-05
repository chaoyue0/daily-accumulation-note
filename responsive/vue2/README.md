## 响应式
### 背景
在没有使用框架时，html与js是分开的，也就是说页面上的元素和变量的值是分开的(`数据和视图分离`)，需要特定的操作才能使变量的变化更新到页面上；

初始化变量的值并将变量挂载到页面上，这属于静态操作，只会在页面DOM节点树初始化渲染时完成，后续修改变量值，DOM节点树不会自动更新；

### 概念
响应式编程是一种`以数据驱动视图变化`的编程模式，它使得系统能够`自动追踪数据的变化并相应地更新`相关的组件、界面或逻辑；

以一种声明式的方式定义数据和数据之间的依赖关系，从而在数据变化时自动更新受影响的部分，而无需显式地编写更新的代码；

### 优点
响应式机制使得开发者能够更专注于数据的变化和处理，而无需手动操作每个受影响的视图，无需手动监听和更新数据变化；


### 数据劫持
既然是以数据驱动视图变化，那么关键在于`如何监听数据`，只要知道数据何时改变并且在数据变化的时候通知视图更新即可；

JS为我们提供了Object.defineProperty方法，可以轻松监听数据，也就是`数据劫持`；

    Vue2响应式系统只会转换数组类型和对象类型的属性

#### Array
原因：JS原生的数组方法(push、pop、shift、unshift、splice、sort、reverse)并不会触发任何通知或回调，导致Vue无法感知到数组的变化；

无法触发响应式变化：

- 直接通过索引修改数组元素的值
- 使用length属性修改数组的长度
- 直接用新数组给旧数组赋值

解决：Vue创建一个拦截器对象，该对象包含了被改写的数组方法，随后将数组的原型对象指向该拦截对象，使数组调用方法时实际上
调用的是改写过的方法(在原生数组方法内加入notifyChange方法，触发响应式更新)。这样就能够劫持对数组的变动，触发相应的更新操作；

```
push(...args) {
  const result = original.push.apply(this, args);
  // 触发响应式更新
  notifyChange();
  return result;
}
```

#### Object
定义：劫持对象中的属性，使用walk方法遍历对象中每对键值，并触发defineReactive进行双向绑定。
如果对象属性仍是对象，那么就会递归劫持内部对象

walk：
````
walk (obj: Object) {
const keys = Object.keys(obj)

    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
}
````

### 依赖关系

#### Dep类
定义：Dep实例是用于管理依赖关系的对象，它与每个响应式对象的属性或数组元素相关联，
每个响应式对象的属性或数组元素都会对应一个独立的Dep实例；

作用：存放 Watcher 观察者对象，建立观察者与被观察者之间的关联；

Dep是一个发布者，可以订阅多个观察者，依赖收集之后Dep中会存在一个或多个Watcher对象，
在数据变更的时候通知所有的Watcher；

#### 属性或元素处理过程

当属性或元素被读取时，触发其getter方法，相关的Dep实例会被存储到全局target变量中，
以建立起观察者与被观察者之间的关联关系，然后Dep实例会将target添加到自身的订阅者列表中；

```allykeynamelanguage
 depend () {
 //依赖收集
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
```

当属性或元素被修改时，触发其setter方法，相关的Dep实例会遍历订阅者列表，通知每个订阅者进行更新操作；

```allykeynamelanguage
 notify () {
 //通知所有订阅者
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
```
    注：为什么要使用slice进行浅拷贝？
    浅拷贝的目的不会为了避免相互影响，而是避免在遍历过程中对原始订阅列表subs进行修改

防止重复添加依赖关系，比如在getter方法中再次访问了响应式属性。
在依赖收集过程中使用Dep.target来跟踪当前的观察者对象


```allykeynamelanguage
Dep.target = null
```