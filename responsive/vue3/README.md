# 响应式系统
## Reactive
作为整个响应式的入口，负责处理目标对象是否`可观察`以及是否`已被观察`的逻辑，最后使用Proxy进行目标对象的代理
### 第一层 reactive
1、判断目标对象的可读性，如果是只读的数据，则直接返回目标对象

2、否则调用createReactiveObject创建observe

### 第二层 createReactiveObject
五个参数：target、isReadonly、baseHandlers(基本类型的Handlers)、collectionHandlers(set、map类型的Handlers)、proxyMap(WeakMap)

1、判断target是否是对象

2、判断target是否是Proxy，如果是Proxy，则直接返回它

3、判断是否有相关联的Proxy，如果有，则返回相关联Proxy

4、判断target是否可被观察，如果targetType为invalid，表示不可观察，直接返回它

5、通过上述的四种判断，避免重复创建Proxy，然后通过new Proxy创建proxy对象并返回proxy

```allykeynamelanguage
const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  proxyMap.set(target, proxy)
```

#### ReactiveFlags
定义：枚举对象，用于表示在响应式系统中标记数据的不同含义，帮助框架跟踪对象的状态、处理代理、只读对象等特征

- SKIP = '__v_skip'： 跳过响应式处理的标记
- IS_REACTIVE = '__v_isReactive'： 表示对象是一个响应式对象的标记
- IS_READONLY = '__v_isReadonly'： 表示对象是一个只读的响应式对象的标记
- RAW = '__v_raw'： 原始对象的标记，用于获取原始未代理的对象
- REACTIVE = '__v_reactive'： 存放响应式对象的标记
- READONLY = '__v_readonly'： 存放只读的响应式对象的标记
- PROXY = '__v_proxy'： 存放代理对象的标记
- RAW_KEYS = '__v_rawKeys'： 存放原始对象的键的标记
- REACTIVE_IS_REACTIVE = '__v_isReactive_reactive'： 用于检测对象是否为响应式对象的标记
- REACTIVE_IS_READONLY = '__v_isReadonly_reactive'： 用于检测对象是否为只读的响应式对象的标记
- IS_SHALLOW = '__v_isShallow'： 用于检测是否为浅层响应式对象的标记
- IS_REF = '__v_isRef'： 用于检测对象是否为 ref 对象的标记
- UNWRAP_REF = '__v_unwrapRef'： 用于获取 ref 对象的原始值的标记
- IS_PROXY = '__v_isProxy'： 用于检测对象是否为代理对象的标记

### 第三层 baseHandlers
#### createArrayInstrumentations
定义：用于劫持Array方法的函数，主要包含两部分逻辑，分别针对某些Array方法进行劫持
```allykeynamelanguage
const instrumentations: Record<string, Function> = {}
```
##### 劫持改变数组长度的方法(push\pop\shift\unshift\splice)
在调用原始Array方法之前`暂停依赖收集pauseTracking`，然后运用原始方法，最后`重新启动依赖收集resetTracking`。这是为了避免在改变数组长度时
引起无限循环的问题

无限循环问题

1、Vue3的响应式系统会在改变数组长度时触发`依赖更新`(这是为了确保数据的变化能够得到及时的响应从而更新视图)

2、导致数组长度变化，这些数组方法也可能会在响应式更新的过程中被`重新调用`(这是为了获取最新值)

3、调用这些方法会触发`新的依赖更新`，如此循环

- 暂停依赖收集pauseTracking：在调用方法之前，先调用pauseTracking方法暂停依赖收集，这样数组改变时不会触发依赖更新，避免无限循环问题
- 重新启动依赖收集resetTracking：在调用方法之后，再调用resetTracking方法重新启动依赖收集，保证数组的响应式特性

##### 劫持不改变数组长度的方法(include\indexOf\lastIndexOf)
为每个方法创建劫持函数，将函数添加到instrumentations对象中

1、劫持函数的作用是调用原始Array方法之前，先对可能是响应式数据的数组进行依赖收集track

2、然后运行原始方法，如果返回-1或false，则再次以非响应式数据调用一次原始方法，以避免可能在原始方法中使用的是响应式数据

        使用非响应式数据调用原始方法相当于绕过了响应式系统的追踪和更新，确保了方法执行的稳定性和准确性。

#### createGetter
两个参数，均为布尔值：isReadonly、shallow，默认都为false

封装了get方法,该方法用于拦截对象的读取属性操作，三个参数：target(目标对象)、property(被劫持的属性名)、receiver(Proxy)

1、判断劫持的属性是否已经是响应式、可读性、浅响应式

2、判断代理的receiver是否为原始对象，再根据代理对象是否只读以及是否是浅层代理，获取当前代理对象所对应的原始对象与receiver进行对比
