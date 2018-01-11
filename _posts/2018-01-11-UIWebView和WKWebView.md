---
layout: post
title: UIWebView和WKWebView的一些比较
category: iOS
date: 2018-01-11 18:00:00 +0800
tags: [学习]
---
介绍一下UIWebView和WKWebView

# 楔子
在我们使用UIWebView的过程中应该有发现UIWebView可是个吃内存的大户，动辄就是上百兆的内存，虽说现在iPhone的性能越来越好，但是也经不住如此折腾啊，况且，一个APP如果占用的系统内存超过整机内存总数的一半时是会被优先kill掉的。
WKWebView 是苹果在 WWDC 2014 上推出的新一代 webView 组件，用以替代 UIKit 中笨重难用、内存泄漏的 UIWebView。WKWebView 拥有60fps滚动刷新率、和 safari 相同的 JavaScript 引擎等优势。

下面就简单讲讲

# UIWebView的内存问题以及一些改进方式
部分App（好吧，可能说很多）喜欢将UIWebView的frame根据内容自适配，因为他们需要在下面添加一些自定义的view，如tableview（需要滚动）等。这种方式正常情况下是没有什么问题的，但是需要考虑一下一些变态情况，如：
 >http://m.thepaper.cn/wifiKey_detail.jsp?contid=1913999&from=wifiKey&fromId=1966555810873344&newsId=26~2490337347993600&docId=26~249033743474688
这个link如果全展开的话，高度超过30000px，最要命的是，如果将webview也进行展开的话，内存直接会飙升到500MB左右，如果说只是将其frame固定，内容滚动的话，比如下图的样式：

![]({{ site.url }}/assets/images/posts/webview_1.jpg)


