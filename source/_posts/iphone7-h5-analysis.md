categories: Note

tags:

- Web

date:  2016-09-08

toc: true

title: iPhone 7 官网展示页分析
---

iPhone 7 今夜闪亮登场，[传送门在此](http://www.apple.com/cn/iphone-7/)。下面是从前端的角度对这个展示页面的简要分析（本文配合官网页面阅读效果更佳）。

<!--more-->

## 动效
在 iPhone 5C 时代登峰造极的 “整屏滚动” 和 “时序错开渐显” 动画被完全弃用了。页面可以非常流畅地滑动至底部，没有通过 `@keyframes` 属性来设置某些产品元素的出现顺序。

整个页面中的动效主要有三类：

1. 首屏 iPhone 7 的图片的视差滚动效果。这个效果应用了 CSS 中的 `transform: matrix3d` 属性来实现平滑的平移效果，可以不依赖 JS.
2. 亮黑色产品质感的展现。例如在摄像头部分，当上下滚动屏幕时，亮黑色的高光和阴影位置会动态改变。这个效果没有通过 CSS 来达成，而是直接在图片上叠加了一层 `canvas` 来实现的。
3. 扬声器介绍页面上的波纹效果，这个水波特效也是在 `canvas` 中绘制的。从加载的脚本资源上来看，它和上面的特效应该都是应用了 `three.js` 这一著名的 WebGL 库来绘制 canvas 的。这样除了躲开低效 CSS 动画的坑以外，还能在低端浏览器上实现优雅降级。


## 布局
相对定位和绝对定位结合的布局，没有应用花哨的 CSS 属性或框架。除文字介绍的偏移量单位采用 `em` 来代替 px 外，对图片的参数设置还是基于 `px` 的。

响应式布局的断点有两个，分别位于 `735px` 和 `1068px` 处。在 CSS 中苹果通过 `@media` 显式区分了不同 Viewport 宽度条件下对应的图片 URL 地址。


## 请求与加载
根据浏览器 UA 的不同，页面的响应区分了桌面版和移动版两个版本。这两个版本除了响应式的布局以外，所有的交互特性保持了一致。

### 连接
以滚动浏览完整个页面为参考，移动端和 PC 端需要的连接资源如下：

* 移动端 59 个 HTTP 请求，消耗约 5.2MB 流量
* PC 端 95 个 HTTP 请求，消耗约 14.1MB 流量

PC 端比移动端多出的请求集中在 [flow](http://images.apple.com/media/us/iphone-7/2016/5937a0dc_edb0_4343_af1c_41ff71808fe5/overview/hero-b/flow/large/flow/flow_004.jpg) 这个诡异的图片资源上。根据 URL 可以判断出它是首屏图片所在 `section` 所使用的资源，然而其内容在页面中完全没有体现，其作用有待后续深入分析。

### 脚本与样式
移动端和 PC 端使用完全一致的 CSS 和 JS 资源：

* 7 个 JS 文件 328KB，已使用构建工具压缩了变量和对象属性名。
* 10 个 CSS 文件 83KB，主要集中在字体和苹果通用 nav / footer 的样式。页面自有的样式合并在一个 CSS 文件中。

### 图片
图片资源根据 `@media` 属性进行针对不同设备的加载，主要包括：

* 大幅的产品效果图根据屏幕尺寸，选择 small / medium / large 三种分辨率。注意 small 下会根据设备 dpi 选择 2x 高清图，这个图片尺寸是超过普通 13 寸笔记本下 medium 尺寸非高清图片的
* 纯白色的图标资源，未用 SVG 而采用了 PNG 格式。同样地，在小屏幕设备上使用了 `2x` 的高清分辨率

### 字体
字体文件不分移动和 PC 端，都占用了 7 个请求，3.3 MB 的流量，毕竟苹果对字体设计一贯有很高的要求。同时，苹果还引入了多个不同字重的字库来满足排版时粗细对比的需求。

英文字体采用了 Myrial Set Pro 的 Thin 和 Semibold 字重，而中文字体采用了经过裁剪的 HanHei-SC 字体。最后列出 Font Fallback 如下：

``` text
'Myriad Set Pro', 'PingFang SC', 'Helvetica Neue', 'Helvetica', 'STHeitiSC-Light', 'Arial', sans-serif
```

## 总结
苹果的产品介绍页的视觉特效风格已经从炫技过渡到了对产品特性更加精细的展示，并充分应用了 `three.js` / 图片响应式适配等成熟的 H5 技术，且在字体上也有一套成熟的解决方案。最终页面加载的请求数和资源数，对类似展示页的开发也提供了不错的参考。
