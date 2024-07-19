# Echarts

- Series 系列：表示一组数值以及它们映射成的图形，用于将数据映射成图形，包含至少一组数据、一个图表类型以及数据如何映射成图的参数

## 原理

### canvas vs css

- 渲染方面：canvas 通过 js 在一个位图上进行逐像素绘制；css 由浏览器引擎进行优化和渲染
- 开销方面：canvas 每次重绘需要重新计算和绘制整个图像；css 会在浏览器的优化下尝试最小重绘区
- 灵活性：canvas 适用于精细控制图形渲染的场景；css 适用于简单的图形和动画
- 交互性：需要额外的 js 代码来处理用户交互；直接使用 HTML 元素的事件处理程序来实现交互，如 :hover、:active 等伪类

### canvas vs svg

- 渲染方式：svg 基于矢量的绘制技术，使用 XML 描述图形，图形可缩放不失真
- 开销方面：适用于静态图形
- 灵活性：每个图形元素都是独立的 DOM 节点，可以方便访问和修改

### zrender 库

## 分类

### 分时图

### markPoint

标记最值或平均值

#### symbol 自定义图形

svg：通过`path：//`绘制矢量路径，支持颜色、大小动态变化

#### data 多组 markPoint

可以为每一个 markPoint 设置不同的样式效果，如 label、offset、symbolRotate 等

#### node 视频生成

- symbolOffset 属性不生效：echart 版本问题，5.2.1 版本支持

#### 动画

- animation
- animationDuration

### 3D 饼图

### 静态配置项

- type：surface 曲面图
- parametricEquation：曲面方程
  - 角度：起始角度、结束角度
  - 值：计算占比
- grid3D：3d 图的属性
  - 高度：根据值动态计算高度
  - viewControl：动态操作
- xAxis3D、yAxis3D、zAxis3D：三维坐标系
- legend：图例

### 动态配置项

- 图表数据
- 动画：是否旋转
- 颜色

### 矩阵树图

- type：treemap
- label：
  - position：上下左右和想象的位置有所不同，最好使用 inside（上下左右）在内部进行位置排列
  - offset：偏移
  - formatter：标签格式（字符串模板和回调函数）
  - align：水平对齐方式
  - verticalAlign：垂直对齐方式

### 雷达图

- type：radar
- axisName：指示器名称的配置项
- nameGap：指示器名称与指示器轴的距离
- silent：是否静态无法交互
- triggerEvent：标签是否响应和触发鼠标事件
- axisTick：坐标轴刻度
- axisLabel：坐标轴标签

#### series 多系列

series 数组只有一个对象，多系列情况用 data 数组划分

### 排名图

- 柱状图横向排布：将 y 轴作为类目轴 yAxis type: 'category'

- label 位于柱状图上方

  - icon + name：yAxis-axisLabel rich-padding 设置偏移量
  - value：series-labelLayout 手动设置标签的 x，y 坐标

- formatter：不同名次显示不同的效果（不能写表达式）

#### 难点

- icon 的偏移，直接使用 padding 会影响 icon 自身的效果，使用 margin 可以进行左右偏移无法上下偏移

- 制作一张偏移好位置的 svg 大图，但 FFCreator 不支持图片的引入（本质是不支持 new Image 的写法）

- 使用 markPoint，symbol 支持 svg 路径的写法，但是 markPoint 原本的作用是显示极值点或平均值

- 使用 markPoint 的 data，可以手动配置多个标记，配合坐标 coord 可以配置 symbolOffset 属性

### 折线图

### 零点多色

- type：piecewise

- pieces：自定义分段文字以及样式
