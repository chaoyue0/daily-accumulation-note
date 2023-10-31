# 主题

## 技术方案

### link标签动态引入
提前准备好几套css主题样式文件，需要的时候创建`link标签`动态加载到head标签中，或者动态改变link标签的`href属性`

优点：实现了按需加载，提高了首屏加载时的性能

缺点：
- 文件过大或者网络不佳情况下可能会有加载延时，导致样式切换不流畅
- 样式表定义不当，会存在优先级问题
- 主题样式是写死的，后续修改或新增麻烦

### 自定义样式适配

- scss变量
- css变量

#### scss
使用sass模块`sass.map`和`@use`来重构所有的scss变量，通过对所有的scss变量使用@use，解决了由@import造成的重复输出问题

    想要覆盖样式库的样式变量，可以使用@forward和with函数结合，重写你需要的样式即可

##### @use

- 通过@use加载的模块不管被引用了多少次，只会在编译后`输出一次`到css中
- 且`拥有命名空间`，旨在当前的样式表中生效
- 以-或_开头的命名空间被认为是私有的，不会引入其他样式表

#####  @import
@import多次引用到同一个模块中，会`反复输出`到css中

##### @forward

- @forward可以引用另一个模块的所有变量、mixin和函数，将它们作为当前模块的`API暴露`出去，而不会在当前模块增加代码
- 不同于@use，@forward不会给变量添加命名空间
- 使子模块下的成员自动带上前缀，使用`as`子句
- 控制成员是否可见，使用`show / hide`
- 控制变量值，使用`with函数`可以改变模块内部变量的初始值

#### css
使用css变量来重构，动态改变组件内的个别变量

优点：
- 不需要重新加载样式文件，在样式切换时不会卡顿
- 利用var绑定变量，不存在优先级的问题
- 新增或修改方便，仅需修改css变量即可，var绑定样式变量的地方就会自动更换

缺点：首屏加载会牺牲一些时间加载样式资源

### v-bind
在style中可以通过v-bind引入js中定义的变量

原理：给元素绑定css变量，在绑定的数据更新时调用cssStyleDeclaration.setProperty更新css变量值
```
p {
    color: v-bind('theme.color');
  }
```

- 直接使用
- 拼接使用：v-bind(width + 'px') 在css中没问题
- 对象调用：v-bind('span.width')，需要引号包裹才能正常使用
### 动态切换主题
#### 系统主题更换
prefers-color-scheme

- @media：css媒体特性用来检测用户是否将系统的主题色设置为亮色light或暗色dark
- window.matchMedia：js方法对指定的媒体查询字符串解析，判断是否匹配媒体查询

较合适的方法：通过`属性选择器`控制根节点css变量
```typescript
  htmlEl.setAttribute("data-theme", isDarkMode ? "dark" : "light");
```
```CSS
html[data-theme="light"]:root {
  --body-background: #efefef;
  --text-color: #333;
}
```

#### 页面提供主题切换按钮，用户主动切换
需要在HTML节点添加自定义属性，并根据当前主题色通过css变量控制，点击按钮后设置自定义属性值即可
```typescript
buttonEl.addEventListener("click", () => {
  const currentTheme = htmlEl.getAttribute("data-theme");
  const nextTheme = currentTheme === "dark" ? "light" : "dark";

  htmlEl.setAttribute("data-theme", nextTheme);
});
```

#### 通过URL控制当前主题
页面加载后根据URL query参数动态添加到html节点自定义属性，并根据当前主题色通过css变量控制