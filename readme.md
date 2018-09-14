# 页面性能优化

- 加载优化
    - 文本（代码）优化（大小）
        - 压缩代码（minify）
        - 压缩传输文件（Gzip）
        - 减少库的使用（只使用库里的一两个小功能，完全  可以用原生自己开发）
        - 删除无用代码
    - 图片优化（大小）
        - 合适的图片格式
            - PNG：插图、需要透明度的图片
            - JPG：照片
            - GIF：动画（最新的Safari支持src为  video的`<img />`）
        - 删除图片的元数据（metadata）以减少大小
        - 缩放图片（在不需要高清图片的地方）
        - 减少图片质量
        - 压缩图片
    - HTTP 请求优化（频率，在HTTP2中将不再那么重要）
        - 合并图片（雪碧图）
        - 图片/视频懒加载
        - 合并Script/CSS代码
        - `<script>`与`<style>`代替外部文件
    - HTTP缓存
        - Cache-control
            - no-cache 每次请求必须验证缓存是否过期
            - no-store 禁止缓存存储
            - max-age 最大缓存时间（相对时间）
            - must-revalidate 在最大缓存时间过期后  必须重新验证是否过期
        - Expire 缓存过期时间（优先级低于   `max-age`）
        - If-Modified-Since 如果文件自指定日期后修  改过，返回新文件
        - If-None-Match 配合E-Tag，如果没有匹配的   E-Tag，则返回新文件
    - 了解渲染流程
        - [关键渲染路径](#关键渲染路径)
        - [CSS阻塞渲染](#css阻塞渲染)
        - [JavaSctipt阻塞DOM构建与页面渲染](#javasctipt阻塞dom构建与页面渲染)
        - [优化关键渲染路径](#优化关键渲染路径)
    - 使用HTTP/2
- 运行优化
    - JavaScript优化
        - 使用[`requestAnimationFrame`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)代替`setTimeout`或 `setInterval`[<sup>[1]</sup>](#关于requestanimationframe)
        - 将长时间运行的代码移到[Web Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API)内
    - CSS优化
        - 降低选择器复杂度
        - 删除无用的CSS选择器
    - [减少重排（重排需要重新计算元素布局）与避免抖动]（https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing?hl=zh-cn）
    - [减少重绘复杂性](https://developers.google.com/web/fundamentals/performance/rendering/simplify-paint-complexity-and-reduce-paint-areas?hl=zh-cn)

## 关键渲染路径

1. DOM构建（骨架）
    1. 字节解析为位版本
    2. 词法、语法分析
    3. 构建DOM对象
2. CSSOM构建（样式描述）
    1. 字节解析为位版本
    2. 词法、语法分析
    3. 构建CSSOM对象
3. Render-Tree构建（带样式描述的骨架）
    1. 从DOM树根节点便利每个**可见**节点
    2. 对每个节点匹配CSSOM并应用它们
4. Layout（布局）
    1. 从渲染树的根节点开始，遍历节点，输出一个“盒模型”，精确捕获每个元素在视口内的确切位置和尺寸
5. Paint
    1. 绘制各个图层
    2. 合并图层
    3. 显示到屏幕

## CSS阻塞渲染

因为Render-Tree的构建依赖于CSSOM，因此CSS将会阻塞页面的渲染。
使用媒体查询来实现CSS不阻塞渲染。

```html
<link href="style.css"    rel="stylesheet"> <!-- 阻塞渲染 -->
<link href="style.css"    rel="stylesheet" media="all"> <!-- 阻塞渲染 -->
<link href="portrait.css" rel="stylesheet" media="orientation:portrait"> <!-- 根据设备方向决定是否阻塞渲染 -->
<link href="print.css"    rel="stylesheet" media="print"> <!-- 不会阻塞渲染 -->
```

## JavaSctipt阻塞DOM构建与页面渲染

因为JavaScript可以查询、修改DOM及CSSOM，其执行将阻止CSSOM，除非声明为异步，它将组织构建DOM。

> 当 HTML 解析器遇到一个 script 标记时，它会暂停构建 DOM，将控制权移交给 JavaScript 引擎；等 JavaScript 引擎运行完毕，浏览器会从中断的地方恢复 DOM 构建。
>
> 如果浏览器尚未完成 CSSOM 的下载和构建，而我们却想在此时运行修改CSSOM的脚本，会怎样？答案很简单，对性能不利：浏览器将延迟脚本执行和 DOM 构建，直至其完成 CSSOM 的下载和构建。
>
> 默认情况下，所有 JavaScript 都会阻止解析器。由于浏览器不了解脚本计划在页面上执行什么操作，它会作最坏的假设并阻止解析器。向浏览器传递脚本不需要在引用位置执行的信号既可以让浏览器继续构建 DOM，也能够让脚本在就绪后执行；例如，在从缓存或远程服务器获取文件后执行。
>
> ———— [使用 JavaScript 添加交互](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript?hl=zh-cn)

将JavaScript标记为异步`async`将防止其阻塞DOM构建。

## 优化关键渲染路径

优化关键渲染路径手段主要有：

- 分析关键资源，将其删除/延迟/异步加载。
- 减少关键资源的大小。
- 优化其加载顺序。

*关键资源指的是与网页首次渲染相关的资源*

### 优化建议

- 优化JavaScript的使用
    - 尽量异步加载JavaScript资源
    - 延迟解析对首次加载无关的脚本
    - 避免长时间运行的JavaScript
- 优化CSS的使用
    - 将CSS置于`<head>`内，尽早构建CSSOM
    - 避免使用`@import`或内联阻塞渲染的CSS，防止网络时延阻止CSSOM构建以及页面渲染

## 关于requestAnimationFrame
requestAnimationFrame将会在下一次页面重绘之前调用其callback，那么它和微任务以及下一个宏任务之间的执行顺序如何呢。通常宏任务需要等待本次宏任务的微任务执行完成之后执行，看以下代码：

```js
let main = () => {
    console.log('begin')
    requestAnimationFrame(() => console.log('animationFrame'))
    setTimeout(() => {console.log('macro')}, 0)
    setTimeout(() => {console.log('macro')}, 100)
    Promise.resolve('micro').then(result => {console.log(result)})
    console.log('end')
}

main()
```

不同的浏览器会有不同的输出：

### Chrome

```js
begin
end
micro
animationFrame
macro
macro
```

### FireFox & Safari

```js
begin 
end
micro 
macro 
animationFrame 
macro
```

