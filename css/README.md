# Css
## 单位
- rpx：表示把屏幕分成750份，1rpx表示一份；
- px："绝对单位"，将页面按像素进行展示；
- em：相对单位，相对于它的父节点字体进行计算；
- rem：相对单位，相对于根节点字体进行计算；
- %：相对单位，一般情况下是相对于`直接`父节点进行计算；


    注意：    
    1、margin和padding的百分比：在垂直方向和水平方向都是相对于直接父亲元素的 width ，而与父元素的 height 无关；
    2、border-radius的百分比：border-radius的百分比是相对于自身宽度，与父元素无关；
- vw：视口宽度，1vw表示占据视口宽度的1%；
- vh：视口高度，1vh表示占据视口高度的1%；

### px像素
px实质也属于相对单位，相对于的是设备物理像素，屏幕实际像素也就是px和`devicePixelRatio`的相乘，
表示浏览器应使用多少屏幕实际像素来绘制单个CSS像素；
#### window.devicePixelRatio
定义：物理像素与逻辑像素的比例，表示每个css像素对应的物理像素数量；

物理像素：物理屏幕上实际的像素点

逻辑像素：css像素点，用于网页布局和渲染

通常情况下，较小的屏幕设备具有较高的设备像素密度(PPI、DPI)，意味着相同屏幕尺寸下，设备上的像素数量更多。
因为较小的屏幕设备需要在有限的物理空间显示更多的像素，因此需要更高的像素密度，用来提供更细腻和清晰的显示效果；

较小的屏幕设备，window.devicePixelRatio值通常较大，例如2，3；

较大的屏幕设备，window.devicePixelRatio值通常较小，例如1；

## 交互效果

### transition 过渡
#### 定义
可以让属性变化成为一个持续一段时间，而不是立即生效的过程，可以控制开始与结束的状态

#### 参数

- transition-property：支持过渡效果的属性（颜色、盒模型相关属性、字重、透明度）
- transition-duration：过渡的时长
- transition-timing-function：过渡的方式（匀速、加速、减速）
- transition-delay：变换延迟时间

### animation 动画
#### 定义
可以将一个css样式配置转换到另一个css样式配置，包含两部分：描述动画的样式规则、指定动画开始，结束以及中间点样式的关键帧

#### 子属性

- animation-delay：设置延时
- animation-direction：动画运行完后是反向运行还是重新运行
- animation-duration：设置动画一个周期的时长
- animation-iteration-count：设置动画重复次数，infinite表示无限次重复动画
- animation-name：由@keyframes描述关键帧名称

#### 关键帧
@keyframes + animation-name {}

- from - to
- 百分比

### transform
#### 定义
允许旋转、缩放、倾斜或平移给定元素，通过修改css视觉格式化模型的坐标空间来实现的

#### 属性

- translate：水平或垂直方向平移
- scale：水平或垂直方向缩放
- rotate：旋转
- skew：倾斜