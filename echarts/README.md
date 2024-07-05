# Echarts

- Series 系列：表示一组数值以及它们映射成的图形，用于将数据映射成图形，包含至少一组数据、一个图表类型以及数据如何映射成图的参数

## 分时图

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

## 3D 饼图

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

## 矩阵树图

- type：treemap
- label：
  - position：上下左右和想象的位置有所不同，最好使用 inside（上下左右）在内部进行位置排列
  - offset：偏移
  - formatter：标签格式（字符串模板和回调函数）
  - align：水平对齐方式
  - verticalAlign：垂直对齐方式

## 雷达图

- type：radar
- axisName：指示器名称的配置项
- nameGap：指示器名称与指示器轴的距离
- silent：是否静态无法交互
- triggerEvent：标签是否响应和触发鼠标事件
- axisTick：坐标轴刻度
- axisLabel：坐标轴标签

## series 多系列

series 数组只有一个对象，多系列情况用 data 数组划分
