# Echarts

- Series 系列：表示一组数值以及它们映射成的图形，用于将数据映射成图形，包含至少一组数据、一个图表类型以及数据如何映射成图的参数

## 原理

echarts 是如何根据用户设定的 option 进行图表的绘制？

### zrender 库

## 分类

### 分时图

### 期货分时图

#### 早市时间点

- 9：00 - 10；15
- 10：30 - 11；30
- 13：30 - 15：00

#### 夜市时间点

常规时间区间：21：00 - 23：00

特殊区间：

- 原油: 21：00 - 凌晨 2:30
- 有色金属和贵金属：21：00 - 凌晨 1:30

#### 分时线

- 白色曲线：即时成交价格
- 黄色曲线：即时成交平均价格
- 蓝色曲线：涨跌幅，表示与前一个交易日的收盘价比较

#### 问题

1、在相同 y 轴区间内，价格和涨跌幅的变化效果一致，导致两根线可能会重合在一起

2、由于夜市时间点不是固定的，x 轴需要展示全部的时间点

3、两根 y 轴需要建立关联，左测的数据与右侧的数据能够对应上

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

#### 零点多色

- type：piecewise

- pieces：自定义分段文字以及样式

### 阶梯图

- 折线图体现的是数据渐变的过程
- 阶梯图的数据只是在某一个特定时间突然发生的变化，不用考虑时间内的变化
- type：line + step：true（start、middle、end 决定拐点的位置）
