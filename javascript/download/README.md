# 文件下载
## 普通url下载
场景：当后端返回的资源是链接时

### 模拟form表单提交

- 创建表单元素
- 创建input起传参的作用(文件名)
- 插入到body中
- 发送请求
- 移除form表单元素

优点：

- 传统方式，兼容性好，不会出现url长度限制

缺点：

- 无法知道下载的进度
- 无法直接下载浏览器可直接预览的文件，如pdf、png等
### window.open
优点：

- 简单直接

缺点：

- 会出现URL长度限制问题
- 无法直接下载浏览器可直接预览的文件，如pdf、png等
- 不能添加header，无法进行鉴权
- 无法知道下载的进度

## 流文件下载
### a标签download属性
为了解决浏览器直接预览png、jpg、tet等文件，而不提供下载的问题，利用html5新增的`download`属性

    如果尝试下载跨域资源，download的属性就会失效，和不设置download表现一致，直接在浏览器预览文件

跨域解决方案：

- 创建一个XMLHttpRequest的get请求
- 将响应类型改成blob类型
- 将url通过createObjectURL转成blob二进制文件
- a标签设置href和download属性
- 触发send方法

优点：

- 能解决不能直接下载浏览器可浏览的文件
- 可以修改header信息，进行鉴权

缺点：

- 有兼容性问题，尤其是IE

### 利用base64
- 利用`FileReader`构造函数以及`readAsDataURL`方法生成`Data URL`，就是`base64`编码后的url形式
- 在onload事件中创建a标签，将e.target?.result传给href
- 触发a标签的click事件
- 移除a标签
``` typescript
   const blob = new Blob([url]);
  console.log('blob', blob);
  const reader = new FileReader();
  reader.readAsDataURL(blob);
  reader.onload = (e: ProgressEvent<FileReader>) => {
    console.log('e.target.result', e.target?.result);
    const a = document.createElement('a');
    a.setAttribute('href', e.target?.result as string);
    a.setAttribute('download', fileName);
    a.click();
  };
```

> readAsDataURL 方法会读取指定的 Blob 或 File 对象。读取操作完成的时候，readyState 会变成已完成DONE，
> 同时 result 属性将包含一个data:URL 格式的字符串（base64 编码）以表示所读取文件的内容。 
 
> FileReader 对象允许 Web 应用程序异步读取存储在用户计算机上的文件（或原始数据缓冲区）的内容，使用 File 或 Blob 对象指定要读取的文件或数据。

## 文件名

### Content-Disposition
在header中包含一个Content-Disposition字段，其中`filename=`和`filename*=`后面包含文件名，获取文件名只要截取这两个字段值

    filename*=不一定有，在旧版浏览器中或个别浏览器不支持这种形式

缺陷：非全数字英文的文件名，浏览器只支持filename，获取的文件名编码可能会有问题

#### filename
需要后端处理好编码形式，但是对应每个浏览器的不同，解析的情况也不同

#### filename*
现代浏览器支持，为了解决filename的不足，一般是`utf-8`，用`decodeURIComponent`就能解码


### a标签download
前端控制下载文件的文件名，通过设置a标签`download`属性的值就是文件的文件名

## 进度条
### XMLHttpRequest onprogress
下载的时候发出进度通知，在下载过程中，排队的一个任务以触发一个progress的进度事件，大约`50毫秒`触发一次

- progress：表示下载的数据量
- timeStamp：表示当前的时间戳，单位是毫秒
- total：表示文件的总体积
- percentage：根据progress和total可以计算出百分比

            重点在于界面的交互效果优化
