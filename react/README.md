# React

## 特点

- 单向数据流，State --> View --> New State --> New View
- 状态驱动视图
- view = fn(state)模型
- JSX+inline style，`渲染逻辑`和`标签`共同存在与`组件`中，HTML和CSS全部写进JavaScript中
  - 每个React组件都是一个JavaScript函数，它会返回一些标签，后将标签渲染到浏览器上

## JSX

### 定义
是JavaScript的语法扩展，可以很好的描述UI交互的本质形式

### React组件与HTML标签
React 的 JSX 使用大、小写的约定来区分本地组件的类和 HTML 标签

- 渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名
- 渲染 React 组件，只需创建一个大写字母开头的本地变量

### 规则
#### 只能返回一个根元素
如果一个组件包含多个元素，需要使用一个父标签把它们包裹起来，可以使用<div>标签或者`<> 和 </>`来代替

    这个空标签被称为Fragment，不会在HTML结构中添加额外节点

原理：JSX本质上在底层被转化为JavaScript对象，不能在一个函数中返回多个对象，除非用一个数组把它们包起来

#### 标签必须闭合

- 开始标签必须带有闭合标签
- 单标签必须写成自闭合标签

### 使用驼峰命名法给大部分属性命名
JSX中的属性会变成JavaScript对象中的键值对，对对象的命名存在限制要求，如不能包含-或不能使用class保留字(class用className代替)