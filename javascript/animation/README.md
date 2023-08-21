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