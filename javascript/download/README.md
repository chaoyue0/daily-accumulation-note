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
生成Data URL，就是base64编码后的url形式
``` typescript
 const fileReader = new FileReader();
 fileReader.readAsDataURL(this.response);
```