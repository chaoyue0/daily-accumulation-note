## 响应式
### 响应式图像
根据不同的屏幕分辨率以及设备像素比来尽可能选择高分辨率的图片
#### 场景
通过img标签下载的图片，图片原始尺寸和图片质量是固定不变的。如果想要让图片在不同屏幕响应式调整的话就需要调整图片的样式，
但是通过样式调整了图片的尺寸，可能会影响到图片的质量(模糊、失真)；

### 原因
- 可以在不同设备上渲染高质量的图片资源
- 浏览器上更快的图片加载速度
- 加载合适的图片

#### 图片类型
- 位图：由像素点组成，一个像素点显示一个颜色；
- 矢量图：由几何图片(线条、曲线)组成；

优缺点：
- 色彩表现：位图色彩表现丰富，常用于展示色彩鲜明、元素较多的大图；矢量图常用于展示logo；
- 体积大小：位图要展示更多的色彩，因此体积较大；
- 适配性：位图和屏幕显示是一对一显示，而矢量图和屏幕显示是一对多显示。矢量图记录了细节繁多的图像信息，设计之初并不知道要匹配何种屏幕；

#### 图片质量
- 模糊：指的是图片在放大或缩小过程中失去了清晰度和细节，表现为边缘、细节不清晰或柔化效果；
- 失真：指的是图片在处理过程中发生了形变，导致图片几何形状、比例和色彩出现了错误；

#### 解决方案
##### 准备工作
1、分析图片需要在多少种设备、多少种屏幕尺寸上显示

2、设计好图片在不同情况下的最佳显示比例和图片类型

3、 找UI设计拿到一组在对应情况下的图片

##### 涉及技术点
- srcset属性


定义：包含一个或多个源图的图片容器，不同源图用逗号分隔，每一组源图由两部分组成(图片URL、x或w描述符)；

图片URL：相对路径或绝对路径，包含图片的后缀名；

描述符：
- x像素比：表示屏幕密度(`Pixel Density`)，window.devicePixelRatio属性可以获取到x的值，默认为1x；
- w像素宽：表示图片原始宽度，`w像素宽 = 图片寬度 (pixel) * Pixel Density`;

w像素宽，拖动浏览器窗口，图片能从设置的小屏到大屏进行图像替换，但大屏往小屏变化，不会进行图片替换；


    局限性：srcset属性只适合显示区域一样大小的图像，如果希望不同尺寸的屏幕，显示不同大小的图像

- size属性：不同设备的图像显示的宽度。`size = 图片寬度 (pixel) * Pixel Density`；

为何需要size属性，有css样式不就够了吗？

需要理解浏览器资源的载入顺序：下載 HTML -> 解析 HTML -> 下載 CSS -> 解析 CSS。因此如果在HTML中定义sizes，可以
在下载css之前，就预先知道图片的实际宽度，决定该下载哪个版本的图片，避免不必要的下载；

图片的宽度还是需要在css中定义

sizes只是提示浏览器预计大小，帮助浏览器决定该下载哪一张图片，并不能控制图片的显示大小，而实际图片的缩放样式还是得依靠css；

    注意：size属性必须与srcset属性w描述符搭配使用，单独使用size属性或与x描述符是无效的；

- window.devicePixelRatio属性：返回当前设备的物理像素与CSS像素的比率，
  浏览器应使用多少屏幕实际像素来绘制单个CSS像素，可以获取图片质量是1x还是2x；
- picture标签、source标签：更精细地控制要显示的圖片，例如小屏幕和大屏幕显示不同的图片集，並各自针对两种不同的图片设置高清和一般的版本


    局限性：用srcset能指定不同大小或分辨率的图像，并根据设备的特性选择最适合的图像进行显示，sizes属性定义不同视口宽度下的图像大小
##### 操作流程
```
<picture>
  <source media="(max-width: 500px)" srcset="cat-vertical.jpg">
  <source media="(min-width: 501px)" srcset="cat-horizontal.jpg">
  <img src="cat.jpg" alt="cat">
</picture>
```
浏览器按照source标签出现的顺序，依次判断设备是否满足媒体查询，如果满足就加载srcset属性指定的图片，并且不再执行后续的source标签和img标签