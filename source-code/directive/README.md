# 指令

## v-if
### 原理
- 当 v-if 指令对应的 value 为 false 的时候会预先创建一个注释节点在该位置
- 在 value 发生变化时，命中派发更新的逻辑，对新旧组件树进行 patch，
- 从而完成使用 v-if 指令元素的动态显示隐藏

### 特点

- 动态向DOM数中添加或者删除DOM元素
- 存在局部编译和卸载的过程
- 惰性计算，若初始条件为假则不进行编译
- 具有更高的性能开销
- 适用于条件不经常变化的场景

### 源码分析
```typescript
// _ctx：组件实例的上下文
render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_ctx.visible)
    ? (_openBlock(), _createBlock("div", { key: 0 })) // 创建真实节点
    : _createCommentVNode("v-if", true) // 创建注释节点
}
```
#### 注释节点
在 Vue 中很多地方都运用了注释节点来作为占位节点，
其目的是在不展示该元素的时候，标识其在页面中的位置，以便在 patch 的时候将该元素放回该位置

#### patch
v-if 订阅了visible变量，当 visible 变化的时候，则会触发派送更新，即 Proxy 对象的 set 逻辑，最后命中 componentEffect 的逻辑

#### componentEffect

