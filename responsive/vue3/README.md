# 响应式系统
## Reactive
作为整个响应式的入口，负责处理目标对象是否`可观察`以及是否`已被观察`的逻辑，最后使用Proxy进行目标对象的代理
### 第一层 reactive
1、判断目标对象的可读性，如果是只读的数据，则直接返回目标对象

2、否则调用createReactiveObject创建observe

### 第二层 createReactiveObject
五个参数：target、isReadonly、baseHandlers(基本类型的Handlers)、collectionHandlers(set、map类型的Handlers)、proxyMap

1、判断target是否是对象

2、判断target是否是Proxy，如果是Proxy，则直接返回它

3、判断是否有相关联的Proxy，如果有，则返回相关联饿Proxy

4、判断target是否可被观察，如果targetType为invalid，表示不可观察，直接返回它

5、通过上述的四种判断，避免重复创建Proxy，然后通过new Proxy创建proxy对象并返回proxy