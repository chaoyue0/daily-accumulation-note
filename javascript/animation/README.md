# 动画
## requestAnimationFrame
### 背景
早期的动画效果通过定时器控制动画的执行，但是无法确保何时能让浏览器把下一帧绘制出来，浏览器的计时器精度不足毫秒，
如Chrome的计时器精度为4毫秒，意味着0-4范围内的任何值要么是0，要么是4，不可能是别的数

### 使用
接收一个`callback`参数，此参数是一个要在`重绘屏幕前`调用的`函数`，表示修改DOM样式以反映下一次重绘有什么变化的地方

    使用requestAnimationFrame方法只会调用一次传入的函数，每次更新用户界面时需要再次手动调用一次，同时需要控制动画何时停止

`window.requestAnimationFrame(callback)`

#### callback函数
参数是一个DOMHighResTimeStamp的实例，该参数与performance.now()的返回值相同，表示requestAnimationFrame方法开始执行回调函数的时刻

#### 返回值
与setTimeout方法类似，返回一个请求ID，可以用于通过另一个cancelAnimationFrame方法来取消重绘任务

### cancelAnimationFrame
`window.cancelAnimationFrame(requestID)`



### 节流
支持requestAnimationFrame方法的浏览器实际上暴露出作为`钩子`的回调队列，这个回调队列是一个可修改的函数列表，包含应该在`重绘之前调用的函数`，
每次调用requestAnimationFrame方法都会在队列上推入一个回调函数，队列的长度没有限制

    钩子：指的是浏览器在执行下一次重绘之前的一个点

#### 原理
通过requestAnimationFrame方法递归地向队列中加入回调函数，可以保证每次重绘最多只调用一次回调函数

## Canvas 画布
使用canvas标签，width和height属性是必填项，用于告诉浏览器在多大面积上绘图，在标签内部的内容是后备数据，表示当浏览器不支持canvas标签时显示的内容

像素化：canvas是基于像素的绘制方式，放大或缩小会影响像素的显示

### 基本的画布功能
#### getContext
定义：返回canvas的上下文，如果上下文没有定义则返回null

参数：contextType表示上下文类型，可能是’`2d`'表示二维渲染上下文、‘`webgl`’表示三维渲染上下文对象等

    在使用canvas之前需要对浏览器进行兼容性处理，判断getContext是否返回null

#### toDataURL
定义：可以导出canvas元素上的图像

参数：生成图片的MIME类型(image、png等)

```typescript
 // 取得图像的数据 URI
 let imgURI = drawing.toDataURL("image/png");
```

### 2D绘图上下文
定义：提供了绘制2D图形的方法，包括矩形、弧形和路径

坐标：坐标原点(0,0)在canvas元素的左上角，所以坐标值都相对于该点计算，x坐标`向右`增长，y坐标`向下`增长

    默认情况下，width和height表示两个方向上像素的最大值

#### 填充和描边
填充：以指定样式(颜色、渐变或图像)自动填充形状，`fillStyle属性`

描边：只为图形`边界`着色,`strokeStyle属性`

#### 绘制矩形
以下三个方法都接收4个参数：x坐标、y坐标、矩形宽度和矩形高度，参数的单位都是像素
##### fillRect
定义：用于以指定颜色在画布上绘制并填充矩形，填充的颜色使用fillStyle属性指定
```typescript
context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);
```

    fillStyle属性的定义必须在fillRect定义的前面
##### strokeRect
定义：用于以指定颜色在画布上绘制矩形轮廓，轮廓颜色使用strokeStyle属性指定

    strokeStyle属性的定义必须在strokeRect定义的前面

##### clearRect
定义：可以擦除画布中某个区域，用于把绘图上下文中的某个区域变透明

#### 绘制路径
绘制的路径是矢量路径，可以通过调整路径的缩放来实现无损缩放

要绘制路径，首先必须调用`beginPath()方法`以表示要开始绘制新路径，具体调用以下方法来绘制路径：

- arc(x,y,radius,startAngle,endAngle,counterclockwise)：以坐标(x,y)为圆心，以radius为半径绘制一条弧线，startAngle表示`起始弧度`，endAngle表示`结束弧度`
counterclockwise表示是否逆时针计算起始弧度和结束弧度，默认为顺时针

    
    一个圆周为2Π弧度，即180度 = Π 弧度
- lineTo(x,y)：绘制一条从`上一点`到(x,y)的直线
- moveTo(x,y)：不绘制线条，只会把绘制光标移动到(x,y)
- rect(x,y,width,height)：以给定宽度和高度在坐标点(x,y)绘制一个矩形，它创建的是一条路径，而不是一个独立的图形
- isPointInPath(x,y)：接收x轴和y轴坐标作为参数，用于确定指定的点是否在路径上，可以在关闭路径前随时调用

创建路径后，可以使用`closePath()`方法绘制一条返回起点的线

最后需要使用`stroke()`方法描画路径

#### 绘制文本
- fillText：实心填充的文本，绘制的文本会被`填充`为指定的颜色，通过设置`fillStyle属性`来指定填充颜色
- strokeText：绘制轮廓的文本，绘制的文本会被`描边`为指定的颜色，通过`strokeStyle属性`来指定描边颜色

这两个方法接收4个参数：要绘制的字符串、x坐标、y坐标和最大像素宽度(可选)

这两个方法最终绘制的结果取决于一下3个属性：
- font：以css语法指定的字体样式、大小、字体族
- textAlign：指定文本的对齐方式
- textBaseLine：指定文本的基线