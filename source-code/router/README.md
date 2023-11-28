# 路由
## 路由模式
### Hash模式
在实际URL之前使用了一个哈希字符(#)，这部分URL从未被发送到服务器，所以不需要服务器层面进行任何特殊处理

#后面hash值的改变，并不会重新加载页面，同时hash值的变化会触发`hashchange事件`

### History模式
改变url之后页面不会重新刷新，也不会带有#号，url改变会触发`popState事件`

## 创建Router实例
### 创建match匹配函数
createMatcher(options.routes || [], this)

- 根据用户路由配置对象生成的path来对应路由记录
- 根据name来对应路由记录的map

### 根据配置的mode创建路由实例

- new HTML5History：创建hash模式
- new HashHistory：创建history模式
- new AbstractHistory：表示不是浏览器环境