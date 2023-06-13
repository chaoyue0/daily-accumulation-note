## DOM Node

### Virtual DOM

#### 定义
用js对象描述一个DOM节点，这个js对象就是这个真实DOM节点的虚拟DOM节点，
且包含真实DOM节点的所需要的一系列属性

#### 背景
Vue是数据驱动视图的，数据发生变化视图也随之发生变化。
视图发生变化时就需要操作真实DOM节点，但是此操作是非常耗时的

#### 解决方案
用js的计算性能来替换操作DOM所消耗的性能，对比数据变化前后的状态，
计算出视图中哪些部分需要更新，只更新修改的部分，这样就可以尽可能的减少DOM节点的操作

#### 分类

- 注释节点

    - text属性：字符串类型，表示具体的文本信息
    - isComment属性：布尔类型，标注一个节点是否为注释节点
- 文本节点

    - text属性：字符串类型，表示具体的文本信息
- 元素节点

    - tag属性：描述节点标签
    - data对象属性：包含class、attribute等属性
    - children数组属性：子节点的信息
- 克隆节点：把已有节点的属性全部复制到新节点中

    - isCloned属性：布尔类型，标注一个节点是否为克隆节点
- 组件节点

    - componentOptions属性：组件的option选项，包含props等
    - componentInstance属性：当前组件节点对应的Vue实例
- 函数式组件节点

    - fnContext属性：函数式组件对应的Vue实例
    - fnOptions属性：组件的option选项

#### Diff算法
对比节点的过程也叫做patch过程

patch：以新的VNode为基准，改造旧的oldVNode使之成为跟新的VNode一样

##### 创建节点
新VNode有，旧VNode没有，就在旧的VNode中创建，并insert插入到DOM中

只有三种类型可以被创建并插入到DOM中，分别为元素节点、文本节点、注释节点

判断节点类型：是否有tag属性判断是否为元素节点，isComment属性值判断是否为注释节点

##### 删除节点
新VNode没有，旧VNode有，就在旧的VNode中删除

##### 更新节点
新VNode有，旧VNode也有，就以新的VNode为准，更新旧的VNode

1、VNode和oldNode是完全相同的两个节点，直接跳出程序

2、VNode和oldNode均为静态节点，直接跳出无需处理

3、VNode是文本节点，VNode和oldNode的文本不同，用VNode的文本替换真实DOM中的内容

4、VNode是元素节点：

- 都有子节点，VNode和oldNode的子节点不同，更新子节点
- 只有VNode有子节点，oldNode是文本节点，则清空DOM中的文本，把VNode的子节点添加到DOM中
- 只有oldNode有子节点，清空DOM中的子节点