---
layout: post
title: UIWebView和WKWebView的一些比较
category: iOS
date: 2018-01-11 18:00:00 +0800
tags: [学习]
---
介绍一下UIWebView和WKWebView

## 楔子
在我们使用UIWebView的过程中应该有发现UIWebView可是个吃内存的大户，动辄就是上百兆的内存，虽说现在iPhone的性能越来越好，但是也经不住如此折腾啊，况且，一个APP如果占用的系统内存超过整机内存总数的一半时是会被优先kill掉的。
WKWebView 是苹果在 WWDC 2014 上推出的新一代 webView 组件，用以替代 UIKit 中笨重难用、内存泄漏的 UIWebView。WKWebView 拥有60fps滚动刷新率、和 safari 相同的 JavaScript 引擎等优势。

下面就简单讲讲

## UIWebView的内存问题以及一些改进方式
部分App（好吧，可能说很多）喜欢将UIWebView的frame根据内容自适配，因为他们需要在下面添加一些自定义的view，如tableview（需要滚动）等。这种方式正常情况下是没有什么问题的，但是需要考虑一下一些变态情况，如：
 >http://m.thepaper.cn/wifiKey_detail.jsp?contid=1913999&from=wifiKey&fromId=1966555810873344&newsId=26~2490337347993600&docId=26~249033743474688
 
这个link如果全展开的话，高度超过30000px，最要命的是，如果将webview也进行展开的话，内存直接会飙升到500MB左右，如果说只是将其frame固定，内容滚动的话，比如下图的样式：
![]({{ site.url }}/assets/images/posts/webview_1.jpg)

此时的内存大小如下图所示:
![]({{ site.url }}/assets/images/posts/webview_2.jpg)
其实刚进去的时候，内存很小，大概几十兆，此时webview是白屏，等load完就是图一的大小了，可见UIWebView确实是内存大户。

```objective-c
    int cacheSizeMemory = 4*1024*1024; // 4MB
    int cacheSizeDisk = 32*1024*1024; // 32MB

    NSURLCache *sharedCache = [[NSURLCache alloc] initWithMemoryCapacity:cacheSizeMemory diskCapacity:cacheSizeDisk diskPath:nil];
    [NSURLCache setSharedURLCache:sharedCache];
```
这段代码相信大家在很多 ` AppDelegate ` 里应该都看过，能起到一定作用，但是效果不是非常大。
这里需要注意一下 `diskPath:nil` 经过和我同事的讨论，是使用默认的path去建立disk缓存，如果你使用自己的path，系统会递归去命中缓存，不过这块我还需要自己去实践一下，待考。

**下面提供另一种思路：**
其实大家想将webview展开到全尺寸，无非就是想让下面的view能够实现自己的业务逻辑，以uitableview为例。实际上，我们可以将tableview进行全展开，虽然也会有些内存浪费，当时绝对比webview全展开来的少，毕竟tableview更加成熟。

其实webview之所以能够滚动，也是因为内部有UIScrollView。

![]({{ site.url }}/assets/images/posts/webview_3.jpg)

那么很自然的，我们可以取到这个srollview，将自己的view添加到这上面不就可以了么。

```swift
    self.webView = web;
  
    NSString *urlStr = @"http://m.thepaper.cn/wifiKey_detail.jsp?contid=1913999&from=wifiKey&fromId=1966555810873344&newsId=26~2490337347993600&docId=26~249033743474688";
    
    NSString *encodedString = [urlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    
    NSURL *weburl = [NSURL URLWithString:encodedString];
    
    NSURLRequest *request = [[NSURLRequest alloc] initWithURL:weburl];
    
    [web loadRequest:request];
    
    web.delegate = self;
    
    UIScrollView *myScrollView = self.webView.scrollView;
    
    UIView *containerView = [UIView new];
    
    [myScrollView addSubview:containerView];
    
    [containerView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(myScrollView);
        make.width.mas_equalTo([UIScreen mainScreen].bounds.size.width);
        make.height.mas_equalTo(defaultTableH);
    }];
    
    self.containerView = containerView;
    
    UITableView *tableview = [UITableView new];
    
    [self.containerView addSubview:tableview];
    
    self.tableView = tableview;
    self.tableView.scrollEnabled = NO;
    
    tableview.delegate = self;
    tableview.dataSource = self;
    tableview.showsVerticalScrollIndicator = NO;
```
由于scrollview的autolayout比较特殊，所以这里采用了拉出一个containerView的原因，具体参见：

>http://mokagio.github.io/tech-journal/2015/06/24/ios-scroll-view-and-interface-builder.html

这里tableview只是添加上去，真正布局在下面的方法里：


```swift
- (void)webViewDidFinishLoad:(UIWebView *)webView {
    //获取页面高度（像素）
    NSString * clientheight_str = [webView stringByEvaluatingJavaScriptFromString: @"document.body.offsetHeight"];
    float contentH = [clientheight_str floatValue];

    [self resizeWebContentHeight:contentH];
    
    @weakify(self);
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        @strongify(self);
        defaultRowCount = 20;
        defaultRowHeight = 60;
        [self.tableView reloadData];
        
        [self resizeWebContentHeight:contentH];
    });
}
```

这里的 `dispatch_after` 实际上模拟数据源变好，tableview的刷新过程。
`resizeWebContentHeight` 如下：

```swift
- (void)resizeWebContentHeight:(CGFloat)contentHeight {
    [self.containerView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.height.mas_equalTo(contentHeight+defaultTableH);
    }];
    
    [self.tableView mas_updateConstraints:^(MASConstraintMaker *make) {
        make.width.mas_equalTo(self.containerView.mas_width);
        make.height.mas_equalTo(defaultTableH);
        make.bottom.mas_equalTo(self.containerView.mas_bottom);
    }];
}
```

这样就模拟的整个的滚动事件，贴两张实际效果图，一张是没更新数据源的图片，一张是更新后的样式：

![]({{ site.url }}/assets/images/posts/webview_4.jpg)

以及

![]({{ site.url }}/assets/images/posts/webview_5.jpg)

毫无违和感，滚动条也完美


