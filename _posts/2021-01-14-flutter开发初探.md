---
layout: article
mathjax: true
title: flutter开发初探
category: flutter
date: 2021-01-14 16:00:00 +0800
tags: [flutter]
---

## flutter开发初探

**作为时下最火的跨端技术，虽然现在才能才入局有点晚的感觉，但是本人是喜欢稳定版的，目前1.22.x也已经官方release了，这篇初探就简单记录下，一枚小白的使用心得和入门吧**

## 国际惯例

![flutter_arch]({{site.url}}/assets/images/posts/flutter_arch.jpg)

摆出这张图，还是简单从整体上来先认识了一下什么是 Flutter，否则容易陷入“盲人摸象”的境地。

* **Embedder** 操作系统适配层，提供线程模型，事件循环模型
* **Engine**：和底层OS无关了，一般是渲染层包括了 Skia 图形绘制库、Dart VM、Text 等，其中 Skia 和 Text 为上层接口提供了调用底层渲染和排版的能力
* **Framework**：是一个用 Dart 实现的 UI SDK，从上之下包括了两大风格组件库（iOS和Android）、基础组件库、图形绘制、手势识别、动画等功能

## Flutter绘制

首先是用户操作，触发 Widget Tree 的更新，然后构建 Element Tree，计算重绘区后将信息同步给 RenderObject Tree，之后实现组件布局、组件绘制、图层合成、引擎渲染。

渲染过程中有3棵树比较重要：

*Widget Tree*, *Element Tree*, *RenderObject Tree*

### Widget Tree

基本逻辑单位，是用户对界面 UI 的描述方式。其实**Widget是不可变的**，只是通过重绘来更新`state `

### Element Tree

它是 Widget 的实例化对象，`createElement` 工厂方法来创建Element。

Element Tree 的重新创建和重新渲染的开销会非常大， 所以 Element Tree 到 RenderObject Tree 也有一个 Diff 环节，来计算最小重绘区域。

需要注意的是，Element 同时持有 Widget 和 RenderObject， 但无论是 Widget 还是 Element，其实都不负责最后的渲染，它们只是“发号施令”，真正对配置信息进行渲染的是 RenderObject。

### RenderObject Tree

RenderObject Tree 在 Flutter 的展示过程分为四个阶段：

1. 布局
2. 绘制
3. 合成
4. 渲染

其中，布局和绘制在 RenderObject 中完成，Flutter 采用深度优先机制遍历渲染对象树，确定树中各个对象的位置和尺寸，并把它们绘制到不同的图层上。绘制完毕后，合成和渲染的工作则交给 Skia 处理。

理论上可以直接让Widget和RenderObject通信，不过因为Widget设计为不可变的，但是最终在屏幕上的object不可能一直不变。如果每次改变都去全局渲染object，会损耗大量性能。所以Element实际上是对Widget做了抽象，只将变化的部分通知Render层，由此最大程度去降低重绘区域，提高渲染效率。

### Flutter绘制流程拆解

1. Build
2. Diff
3. Layout
4. Paint
5. Composite
6. Render

### 自绘引擎

1. 通过Skia直接调用OpenGL渲染，保证性能同时抹平差异。
2. Dart同时支持JIT和AOT。

## Flutter混合开发

### 混合模式

1. 统一管理模式

   所谓统一管理模式，就是一个标准的 Flutter Application 工程，而其中 Flutter 的产物工程目录（ `ios/` 和 `android/` ）是可以进行原生混编的工程，如 React Native 进行混合开发那般，在工程项目中进行混合开发就好。但是这样的缺点是当原生项目业务庞大起来时，Flutter 工程对于原生工程的耦合就会非常严重，当工程进行升级时会比较麻烦。因此这种混合模式只适用于 Flutter 业务主导、原生功能为辅的项目。

2. 三端分离模式

   后来 Google 对混合开发有了更好的支持，除了 Flutter Application，还支持 Flutter Module。所谓 Flutter Module，恰如其名，就是支持以模块化的方式将 Flutter 引入原生工程中， 它的产物就是 iOS 下的 Framework 或 Pods、Android 下的 AAR，原生工程就像引入其他第三方 SDK 那样，使用 Maven 和 Cocoapods 引入 Flutter Module 即可。 从而实现真正意义上的三端分离的开发模式。

### 混合栈原理

混合导航栈主要需要解决以下四种场景下的问题：

* Native 2 Native

  这种情况比较简单，Flutter Engine 已经为我们提供了现成的 Plugin，即 iOS 下的 FlutterViewController 与 Android 下的 FlutterView（自行包装一下可以实现 FlutterActivity），所以这种场景我们直接使用启动了的 Flutter Engine 来初始化 Flutter 容器，为其设置初始路由页面之后，就可以以原生的方式跳转至 Flutter 页面了。

* Flutter 2 Flutter

  * 使用 Flutter 本身的 Navigator 导航栈
  * 创建新的 Flutter 容器后，使用原生导航栈

* Flutter 2 Native

  这里的跳转其实是包含了两种情况，一是打开原生页面（open，包括但不限于 push），二是回退到原生页面（close，包括但不限于 pop）。

* Native 2 Native





