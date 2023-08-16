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

3、定义一个res变量，通过Reflect.get方法从一个对象中取属性值返回给res

4、如果对象是非只读属性，那么就对目标对象进行`依赖收集`，追踪对响应式象属性的访问

5、如果对象是浅响应式的，就直接返回res

6、检测res是ref对象时，实现对ref对象的拆包处理

#### createSetter
一个参数，布尔值：shallow，默认都为false

封装了set方法,该方法用于拦截对象的写操作，四个参数：target(目标对象)、key(被劫持的属性名)、value(要修改的值)、receiver(Proxy)

1、判断新值与旧值，在非浅响应式的前提下，先将新旧值都toRaw转成原始类型的值

2、如果target不是数组，旧值为ref对象，新值不是ref对象，则将新值赋值给旧值

3、hadKey变量：判断target是否包含要设置的属性，如果target是数组且key为整数，判断key是否小于数组的长度；否则通过hasOwn判断是否包含属性

4、result变量：Reflect.set方法来设置target的key属性值为value

5、如果target不包含key属性，说明这是一个`添加属性`的操作，触发`trigger函数`并通知依赖此对象的副作用函数有个属性被添加

6、如果target包含key属性且值发生改变，说明这是一个`更新属性`的操作，触发`trigger函数`并通知依赖此对象的副作用函数有个属性被更新

### 第四层 effect 依赖收集
#### track
作用：访问响应式数据时进行`依赖追踪`，建立响应式数据与副作用函数之间的关联

参数：target响应式对象、type定义响应式对象的类型、key跟踪响应式对象的属性

1、如果target在`targetMap`结构中没有与target对应的依赖集合，表示是`第一次访问这个响应式数据`，需要`创建`一个新的空依赖集合并set到Map结构中

2、如果dep在`DepsMap`结构中没有与key对应的Dep对象，表示是`第一次访问target的key属性`，需要`创建`一个新的空Dep对象(createDep)

3、将dep作为参数调用`trackEffects函数`
##### targetMap和DepsMap
targetMap是`WeakMap对象`，用来存储响应式数据和它们对应的`依赖集合(DepsMap)`之间的关系。
targetMap的键是一个响应式对象，对应的值是depsMap对象。
作用是建立响应式数据与它们依赖集合之间的关联，进行依赖追踪

depsMap是`Map对象`，用于存储响应式数据关联的所有属性的依赖关系。
depsMap的键是一个属性名，对应的值是Dep对象。
作用是建立响应式数据的属性与它们依赖之间的关联，当属性发生变化时，通过`Dep对象触发`该属性相关联的所有副作用函数进行响应式更新

##### createDep
创建一个新的dep，dep是一个set实例，且添加了两个属性

- w：表示当前依赖`是否被收集`
- n：表示当前依赖`是否是新收集的`
```
export const createDep = (effects?: ReactiveEffect[]): Dep => {
 const dep = new Set<ReactiveEffect>(effects) as Dep
 dep.w = 0
 dep.n = 0
 return dep
}
```

#### trackEffects
作用：将当前活动的响应式 effect（即正在执行的副作用函数）与相关的依赖关系建立关联。
在响应式数据被访问（被读取）时，系统就能够知道哪些副作用函数依赖于这个数据，从而在数据变化时，能够通知这些副作用函数进行更新。

1、根据shouldTrack判断依赖是否需要被捕获，默认shouldTrack的值为false，表示不需要被捕获

2、effectTrackDepth表示依赖深度，小于依赖追踪的最大深度情况下，判断是否需要进行依赖追踪，如果需要则将shouldTrack设置为true

3、将当前活跃的effect添加到dep中，同时将dep加入到受副作用影响的依赖集合中 activeEffect!.deps

### 第四层 effect 触发更新
#### trigger
作用：触发副作用函数的更新，在响应式数据发生变化时，通知相关联的副作用函数进行响应式更新

##### 操作类型
```
export const enum TriggerOpTypes {
  SET = 'set', // 设置操作，将旧值设置为新值
  ADD = 'add', // 新增操作，添加一个新的值 例如给对象新增一个值 数组的 push 操作
  DELETE = 'delete', // 删除操作 例如对象的 delete 操作，数组的 pop 操作
  CLEAR = 'clear' // 用于 Map 和 Set 的 clear 操作。
}
```

##### 步骤
1、声明一个deps数组，用于储存与目标对象target和操作类型type相关联的所有Dep对象

2、当目标对象是数组且修改的是数组的长度时，需要触发`与数组长度相关的副作用函数`的更新

3、当操作类型是clear时，表示目标对象被清空，需要触发`所有与目标对象相关联的副作用函数`的更新

4、当操作类型是另外集中类型时，需要触发与目标对象的`指定属性key相关联的副作用函数`的更新

5、如果deps中只有一个Dep对象，表示只有一个副作用函数与目标对象相关联，直接触发这个副作用函数的更新`triggerEffects`

6、如果deps中包含多个对象，表示多个副作用函数与目标对象相关联，将这些副作用函数放入`effects`数组中，并通过`createDep(effects)`创建一个新的 Dep 对象，用于触发这些副作用函数的更新
(一个响应式数据可能与多个副作用函数相关联)

    使用createDep的原因：将effects放入Set中去除重复的effect，并初始化dep信息

#### triggerEffects
主要针对`计算属性`进行逻辑处理，计算属性是一个特殊的副作用函数，具有惰性求值的特性，只有在依赖数据发生变化时才会重新计算

##### 写法特点
双for循环，遍历effects数组，按照计算属性与非计算属性的顺序依次触发副作用函数的更新

    保证计算属性的正确更新顺序和依赖追踪的准确性，如果只是在一个for循环中处理可能会导致计算属性和非计算属性的更新顺序混乱，计算属性的依赖追踪出现问题

#### triggerEffect
主要用于控制副作用函数effect的触发时机

1、判断该effect副作用函数与当前执行的副作用函数是否相同，以及该effect是否是否允许自身内部递归调用
```
effect !== activeEffect || effect.allowRecurse
effect主要负责追踪和响应响应式数据的变化，当响应式数据发生变化时，与之相关的副作用函数就会执行。
如果执行副作用函数的过程中引起了响应式数据的变化，从而又会触发副作用函数的执行，
这样就导致副作用函数递归调用，可能会出现无限循环的问题

情况1：effect与当前执行副作用函数相同，表示可能出现递归调用，如果allowRecurse为true，也可以执行条件体内代码
情况2：effect与当前执行副作用函数不相同，不管如果allowRecurse为true还是false，都会执行条件体内代码
```

2、scheduler调度属性，用于控制副作用函数的执行时机，表示副作用函数会延时执行，而不是立即执行

3、如果没有scheduler属性，则直接调用run方法，执行副作用函数的主体逻辑

### 第四层 effect run
1、为了避免已经停止的副作用函数再次执行，需要检查active的状态，如果没有被激活，则直接执行副作用函数

2、使用parent变量追踪当前实例的父级副作用函数，lastShouldTrack变量保存当前依赖追踪的状态

3、为了避免递归调用，使用while循环来检查当前实例是否已存在于执行栈中，避免副作用函数无限循环调用

4、开始依赖追踪，将当前实例设置为当前激活状态，并将shouldTrack设为true，表示当前的实例可有进行依赖收集

5、根据依赖追踪的深度来选择不同的处理方式

- 小于最大深度限制的话则进行依赖追踪标记的初始化，在浅层级的情况下，使用位运算来进行依赖收集，减少内存占用和操作成本
- 大于最大深度限制的话则进行完整的依赖清理操作，在深层级的情况下，确保依赖追踪的正确性

6、执行副作用函数的主体逻辑并返回结果

7、使用finally清理操作和恢复状态，小于最大深度限制则调用`finalizeDepMarkers函数`处理，并将实例的父级副作用函数设为undefined，如果存在延时副作用则调用`stop函数`停止当前实例

### 第四层 effect stop
作用：对effect的的停止和清理操作，确保在正确的时机停止effect的执行，避免不必要的计算和依赖追踪

1、判断当前effect是否正在执行，是的话将deferStop设置为true，表示需要延时处理

2、判断当前effect的激活状态，如果是激活状态，则调用`cleanupEffect函数`

3、如果effect中存在onStop函数，则调用`onStop函数`

4、将effect的激活状态调整成false

#### cleanupEffect

1、对象解构赋值，从effect中提取deps属性

2、遍历deps，删除effect，清理依赖和effect之间的关联关系

3、将deps的length调整成0

### 第五层 Dep
#### finalizeDepMarkers函数
作用：完成了对副作用函数effect的依赖追踪标记的最后处理，根据依赖状态来决定是否保留依赖，并将依赖追踪标记清零，以便下一轮的依赖追踪

1、从参数effect中获取deps数组

2、遍历deps数组，判断是否当前的dep是否曾经被追踪`wasTracked`并且当前不再被追踪`newTracked`，如果是则从dep的依赖列表中删除effect，如果不是则保留在deps数组中(使用ptr表示已有数据的下标)

3、使用位运算符清除当前dep的依赖追踪标记

4、将deps数组的长度截断为ptr，移除多余的空项

    为什么使用位运算符清除：高效的标记与清零、节省内存空间(可以将多个标记位存储在一个整数中)、提高性能(计算机硬件层面的操作)

## Ref
Vue3的响应式是基于Proxy实现的，而Proxy的前提是代理的变量是对象而不能是基本类型的数据，
如果想要对基本类型数据实现响应式就需要将基本类型包装成Ref对象，从而实现响应式

### createRef 函数    
参数：rawValue、shallow布尔值

判断rawValue值是否已经是Ref对象了，如果是则直接返回，如果不是，需要调用`RefImpl类`创建

#### RefImpl 类
使用`class`生成实例对象

1、定义两个私有变量value、rawValue，以及两个公有变量dep、isRef(readonly)

2、通过constructor方法编写构造函数的内容，对实例的两个私有变量进行赋值
```typescript
    this._rawValue = __v_isShallow ? value : toRaw(value)
    this._value = __v_isShallow ? value : toReactive(value)
```
3、拦截get方法，trackRefValue追踪实例对象，并返回value

4、拦截set方法，如果newValue是shallow或者readonly，则直接返回，如果不是通过`toRaw`转成原始值。判断新旧值是否相等(值是否改变)，对实例私有变量进行重新赋值，最后通过`triggerRefValue`触发
```typescript
    this._rawValue = newVal
    this._value = useDirectValue ? newVal : toReactive(newVal)
```

### Ref 类型

#### 简单类型 number / string / boolean / Function
```
function ref<T>(value: T): Ref<T>
```

#### Ref 类型嵌套
期望直接通过a.value获取值，而不是a.value.value.value
##### extends 条件判断
```
function ref<T>(value: T): T extends Ref ? T : Ref<UnwrapRef<T>>
```
针对ref函数的返回值进行条件判断，如果T是Ref类型，则直接返回，否则需要将T包装成Ref类型再返回

##### UnwrapRef 类型(非Ref类型)
表示用`索引签名`的形式，对Ref类型的`反解`，作用：

- 如果范型T是Ref类型，则UnwrapRef类型是ref.value的类型
- 如果范型T不是Ref类型，则UnwrapRef类型是T

```typescript
type UnwrapRef<T> = {
  ref: T extends Ref<infer R> ? R : T
  other: T
}[T extends Ref ? 'ref' : 'other']
```

    还需要考虑UnwrapRef<T>中的T是对象，且这个对象中包含Ref的情况，还需要UnwrapRef进行反解

```typescript
type UnwraRef<T> = {
  ref: T extends Ref<infer R> ? R : T
  // 注意这里
  object: { [K in keyof T]: UnwrapRef<T[K]> }
  other: T
}[T extends Ref 
  ? 'ref' 
  : T extends object 
    ? 'object' 
    : 'other']
```

遍历`K in keyof T`的时候，只要对值类型 T[K] 再进行解包UnwrapRef<T[K]>

#### object / Array 类型

- object：`in keyof` 遍历object的每个键，每个键值不能是Ref类型
- Array：遍历Array的每个元素，元素保留自身的类型，如果元素是object、Array，需要再次进行遍历操作

##### object
```typescript
type UnwrapRef<T> =  T extends BaseTypes | Function ? T : T extends object ? UnwrapObject<T> : T

type UnwrapObject<T> = { [P in keyof T]: UnwrapRef<T[P]> } 
```

    递归调用UnwrapRef，因为可能对象的键也是对象，存在对象嵌套对象的形式

##### Array
```typescript
type UnwrapArr<T> = { [K in keyof T]: UnwrapRefSimple<T[K]> }
```

将之前的UnwrapRef的功能拆分成两部分，一部分由`UnwrapRefSimple`实现，而`infer`的提取功能由`UnwrapRef`实现

#### 总结
ref函数始终返回一个类型为Ref的值

1、参数是`简单类型`，则直接返回

2、参数是`Ref`，用infer v获取参数类型，返回v

3、参数是`object`，遍历object将属性解成简单类型

4、参数是`Array`，遍历Array保留数组元素中的类型

```typescript
type UnwrapRef<T> =  T extends Ref<infer V> ? UnwrapRefSimple<V> : UnwrapRefSimple<T>

type UnwrapArr<T> = { [K in keyof T]: UnwrapRefSimple<T[K]> }

type UnwrapObject<T> = { [P in keyof T]: UnwrapRef<T[P]> }

type UnwrapRefSimple<T> = T extends
    | BaseTypes
    | Function
    | Ref
    ? T
    : T extends Array<any>
        ? UnwrapArr<T>
        : T extends object
            ? UnwrapObject<T> : T
```