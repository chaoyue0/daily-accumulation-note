# FFCreator

## 原理

使用 opengl 来处理图形渲染并使用 shader 后处理来生成转场效果，最后使用 FFmpeg 合成视频

### 对象池

## 使用

- 入口 creator
- 场景 FFScene
- 元素：图片、视频、字幕
- 音乐

- 一个影片可以添加多个场景，场景之间可以设置过渡特效动画和停留时长
- 一个场景可以添加多个不同元素，对元素可以添加各自的动画
- 既可以在整个影片上添加背景音乐，也可以在单个场景上添加声音（适合语音 tts）

### 构造函数创建

#### FFCreator 创建入口

```
const creator = new FFCreator({...配置项})
```

- cacheDir 缓存目录
- outputDir 输出目录
- output：输出文件名
- width
- height
- cover：设置封面
- audioLoop：音乐是否循环
- fps：帧率，表示每秒播放的帧数
- threads：多线程并行渲染

#### FFScene 创建场景

```
const scene = new FFScene();
// 调用方法设置动画效果
...
creator.addChild(scene); // 挂载到入口上
```

- setBgColor：设置背景色
- setTransition：设置停留时长
- setTransition：设置过渡动画（类型，时间）

#### FFImage 创建图片元素

```
const img = new FFImage({path: imgpath});
scene.addChild(img); // 挂载到场景上
```

- setXY：设置位置
- setScale：设置缩放
- setRotate：设置旋转
- setOpacity：设置透明度
- setWH：设置宽高
- addEffect：设置动画效果

#### FFAlbum 创建相册元素

```
const album = new FFAlbum({
  list: [img1, img2, img3, img4], // 相册合集
})
scene.addChild(album);// 挂载到场景上
```

#### FFGifImage 创建 GIF 图

```
const fgirl = new FFGifImage({ path: 'a.gif', x: 300 });
```

#### FFText 创建文本元素

```
const text = new FFText({text: '这是一个文字', x: 250, y: 80});
```

#### FFSubtitle 创建字幕元素

```
const subtitle = new FFSubtitle({
  comma: true,                  // 是否逗号分割
})
```

- setText：设置文案
- frameBuffer = 24; 缓存帧
- setSpeech(dub); 设置语音配音-tts
- setDuration：设置字幕总时长

#### FFVideo 创建视频元素

```
const video = new FFVideo({})
```

- setAudio(true); // 是否有音乐
- setTimes('00:00:43', '00:00:50'); // 截取播放时长

#### FFVtuber 创建虚拟主播

FFVtuber 是基于序列帧的简单版卡通动画, 基于它你可以实现自己更细腻的主播形象

```
const vtuber = new FFVtuber({ x: 100, y: 400 });
creator.addVtuber(vtuber);
```

### 属性或事件添加

#### 添加音乐

- 添加全局背景音：creator.addAudio
- 添加场景背景音：scene.addAudio

##### volume 音量

- 0.5:音量减半
- 1.5:音量为原来的 150%
- 20dB:增加 10 分贝
- -20dB:减小 20 分贝

#### 添加事件监听

- start：实例开始工作
- error：发生错误
- progress：处理过程中
- complete：处理完成

通过 on 监听

- 参数 1：事件名
- 参数 2： 事件回调

### 多任务处理

#### FFCreatorCenter 添加任务

针对多个制作任务，进行任务队列管理

```
const taskId = FFCreatorCenter.addTask(() => {
    const creator = new FFCreator({...});
    ... // 添加各种场景和元素

    return creator;  // 一定要return creator
});
```

#### FFCreatorCenter 任务成功回调

```
FFCreatorCenter.onTaskComplete(taskId, result => {
    console.log(result.file);
});
```

#### FFCreatorCenter 任务失败回调

```
FFCreatorCenter.onTaskError(taskId, error => {
    console.error(error);
})
```

## 动画

### addEffect 添加 effect 动画

可以往元素上添加 effect 动画效果

```
image.addEffect({
  type: 'fadeInDown',  // 可以在https://animate.style/查询
  time: 1,
  delay: 1,
});
```

### createEffect 创建自定义 effect

```
creator.createEffect('customEffect', {
  from: { opacity: 0, y: 350, rotate: 190, scale: 0.3 },
  to: { opacity: 1, y: 200, rotate: 0, scale: 1 },
  ease: 'Back.Out',
});
```

### addAnimate 添加自定义动画

addAnimate 支持从 A 到 B 的缓动动画, 可以设置 from、to 属性

```
image.addAnimate({
    from: { x:200, scale: .1, alpha: 0 },
    to: { x:100, scale: 1, alpha:1 },
    time: 1,
    delay: 1.2,
    ease: 'Quadratic.In'
});
```

### setTransition 设置场景切换特效

```
scene.setTransition({
  name: 'TricolorCircle',
  duration: 1.5,
  params: {},
});
```

## 数据可视化

### FFChart 组件
