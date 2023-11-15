# 性能优化
从过程趋势来看，性能优化可以分为`网络层面`和`渲染层面`；从结果趋势来看，性能优化可分为`时间层面`和`体积层面`

## 网路层面
### 构建策略
主要围绕webpack做相关处理
#### 减少打包时间
##### 缩减范围
配置`include/exclude`缩小Loader对文件的搜索范围，可以避免不必要的编译
##### 缓存副本
配置`cache`缓存，加快构建速度
```
 cache: {
      type: 'filesystem',
      allowCollectingMemory: true,
    },
```
##### 定向搜索
配置`resolve`提高文件的搜索速度,alias映射模板路径，extensions表明文件后缀
```
  resolve: {
    alias: {
        '@':'C:\\Users\\jiayu01\\WebstormProjects\\fba_admin_vue3\\src',
         vue$:'vue/dist/vue.runtime.esm-bundler.js'
    }
,
    extensions: [
        '.tsx',
        '.ts',
        '.mjs',
        '.js',
        '.jsx',
        '.vue',
        '.json',
        '.wasm'
    ],
}
```
##### 并行构建
配置`Thread`将Loader单进程转换为多进程，释放CPU多核并发的优势

使用thread-loader开启多线程
##### 可视结构
配置`BundleAnalyzer`分析打包文件结构，直观分析打包文件的模块组成部分、模块体积占比、模块包含关系、模块依赖关系、文件是否重复、压缩体积对比等可视化数据

#### 减少打包体积
##### 分割代码
将一个大的js文件分割成多个较小的代码块，将公共的代码抽离成单独的chunk，避免资源重复打包，
在需要的时候按需加载 ，并充分利用浏览器的缓存机制，缩短资源加载时间

- module：应用程序的一个单独的模块，webpack将每一个模块视为一个独立的单元，并通过识别模块之间的依赖关系来构建应用程序
- chunk：一组相互依赖的module，组合在一起即可并行加载，浏览器可同时加载多个chunk而不是一个大文件
- bundle：包含一个或多个chunk和其他资源文件

        module、chunk、bundle其实是同一份代码在不同转换场景的三种webpack术语，源文件是module，webpack处理时是chunk，浏览器可以直接运行的是bundle
###### 移除重复的代码块
使用 splitChunks 的 `cacheGroups` 配置：

- reuseExistingChunk属性重用模块，而不是重新生成
- enforce：表示是否强制拆分

###### 将PNPM包拆分为体积适中的chunk
vue cli默认的webpack配置已经完成了此项配置，移除了node_module包中的重复模块
##### 摇树优化
使用`import`导入模块，使用`export`导出模块 

    Tree Shaking在生产环境下默认启动

如果想在开发环境启动Tree Shaking，需要配置optimization.usedExports为true，在 Webpack 编译过程中启动标记功能

原理：它会将每个模块中没有被使用过的导出内容标记为`unused`，当生成产物时，被标记的变量对应的导出语句会被删除
##### 动态垫片
polyfill：降级\替代方案，指可以将ES6+的API转换成在低版本的浏览器上可以实现相同功能的替换实现
##### 按需加载
将路由页面、触发性功能单独打包为一个文件，使用时才加载，减轻首屏渲染的负担
##### 作用提升
##### 压缩资源
###### html压缩
html-wepack-plugin插件，可以压缩html

###### css压缩
optimize-css-assets-webpack-plugin插件，可以压缩css

    如果要在开发环境也进行css压缩，需要将minimize配置属性设为true

###### js压缩
terser-webpack-plugin插件，可以压缩js

    在webpack v5中是开箱即用的，如果需要自定义配置的话需要引入该插件
```
 new TerserPlugin(
        {
          terserOptions: {
            compress: {
              arrows: false,
              collapse_vars: false,
              comparisons: false,
              computed_props: false,
              hoist_funs: false,
              hoist_props: false,
              hoist_vars: false,
              inline: false,
              loops: false,
              negate_iife: false,
              properties: false,
              reduce_funcs: false,
              reduce_vars: false,
              switches: false,
              toplevel: false,
              typeofs: false,
              booleans: true,
              if_return: true,
              sequences: true,
              unused: true,
              conditionals: true,
              dead_code: true,
              evaluate: true
            },
            mangle: {
              safari10: true
            }
          },
          parallel: true,
          extractComments: false
        }
      )
```

### 图像策略
#### 图像选型
了解所有图像类型的特点及其其何种应用场景最合适

| 类型       | 体积 | 质量 | 兼容性 | 请求 | 压缩   | 透明  | 场景                                 |
|----------|----|----|-----|----|------|-----|------------------------------------|
| JPG\JPEG | 小  | 中  | 好   | 是  | 有损压缩 | 不支持 | 色彩丰富图、背景图、头像<br />(不适合含文字图像，影响可读性) |
| PNG      | 大  | 高  | 好   | 是  | 无损压缩 | 支持  | 信息图表以及包含图像和文本的图形、屏幕截图              |
| WebP     | 小  | 中  | 差   | 是  | 有损、无损压缩 | 支持  | JPEG和PNG文件最佳替代格式，可节省带宽并提升网站加载速度    |
| SVG      | 小  | 高  | 好   | 是  | 无损压缩 | 支持  | logo、矢量图，不建议复杂图像                   |
#### 图像压缩
在部署到生产环境前使用工具或脚本对其压缩处理

常用的工具：QuickPicture、TinyPng

### 分发策略
主要围绕`内容分发网络`做相关处理，购买CDN服务

原理：一组分布在各地存储数据副本并可根据`就近原则`满足数据请求的服务器，核心是`缓存`和`回源`

- 缓存:是把资源复制到CDN服务器里
- 回源：资源过期\不存在就向上层服务器请求并复制到CDN服务器里


    所有静态资源走CND：开发阶段确定哪些文件属于静态资源(不常变化的样式文件、脚本文件和多媒体文件，如字体、图像、音频、视频)
### 缓存策略
主要围绕浏览器缓存

## 渲染层面
### css策略

- 避免出现超过三层的嵌套规则
- 避免为ID选择器添加多余选择器
- 避免使用标签选择器代替类选择器
- 避免重复匹配重复定义，关注可继承属性
### DOM策略

- 缓存DOM计算属性
- 避免过多DOM操作
### 阻塞策略

- 脚本与DOM的依赖关系很强，对script标签设置defer
- 脚本与DOM的依赖关系不强，对script标签设置async
### 回流重绘策略

- 使用类合并样式，避免逐条改变样式
- 使用display控制DOM显示隐藏，将DOM离线化，而不是改变DOM树
- 缓存DOM计算属性
### 异步更新策略
在异步任务中修改DOM时，把其包装成微任务(promise、nextTick)

## 优化标准
目的：希望降低程序的`整体开销`，应该把重点放在对程序`整体开销影响最大`的那部分上

原则：评估优先，拒绝任何不能提供良好效益的优化，浏览器通常在运行js上花费的时间很少，绝大部分时间消耗在DOM上

### 性能指标轴线

- 低效线：会降低用户的关注度，并使用户烦躁不安
- 受挫线：用户会意识到自己在被迫等待，导致用户开始考虑其他事情
- 失败线：由于应用崩溃或浏览器产生一个对话框告诉用户应用失败，用户只能刷新或关闭浏览器

### 性能指标时间点
如何才算“足够快”

- 0.1秒：用户直接操作UI中对象的感觉极限
- 1秒：用户随意地在计算机指令空间进行操作而无需过度等待地感觉极限，超过1秒地延时要提示用户计算机正在处理问题，例如loading
- 10秒：用户专注于任务的极限，需要一个百分比完成指示器，以及一个方便用户中断操作的方法

### 测试原则
测试用户环境，使用低端机器和低速网络来测试，在开发人员的高配置环境中得出的测试结果还可能会掩盖性能问题

### 测试延时时间

- 手动代码检查：代码中添加`计时器`记录函数的`执行时间`
- 自动代码检测：通过`工具`进行`性能分析`，查找瓶颈或运行最慢的代码块


    工具：Firefox插件Firebug包含一个js代码性能分析器（已下架）

## 无阻塞加载脚本
背景：script标签的阻塞行为会对页面性能产生负面影响，加载脚本的同时不会下载其他内容

下载和执行脚本出现阻塞的原因：脚本可能会改变页面或js的命名空间，会对后续内容造成影响

    脚本必须按顺序执行，但没必要按顺序下载

## webpack打包速度优化

### 方案
缓存、多进程、抽离、拆分

### 