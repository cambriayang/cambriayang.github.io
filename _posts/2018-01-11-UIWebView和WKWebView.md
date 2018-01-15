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

```swift
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

## WKWebView
简单的适配方法本文不再赘述，主要来说说适配 WKWebView 过程中填过的坑以及善待解决的技术难题。

### WKWebView的白屏问题
WKWebView 自诩拥有更快的加载速度，更低的内存占用，但实际上 WKWebView 是一个多进程组件，Network Loading 以及 UI Rendering 在其它进程中执行。初次适配 WKWebView 的时候，我们也惊讶于打开 WKWebView 后，App 进程内存消耗反而大幅下降，但是仔细观察会发现，Other Process 的内存占用会增加。在一些用 webGL 渲染的复杂页面，使用 WKWebView 总体的内存占用（App Process Memory + Other Process Memory）不见得比 UIWebView 少很多。

在 UIWebView 上当内存占用太大的时候，App Process 会 crash；而在 WKWebView 上当总体的内存占用比较大的时候，WebContent Process 会 crash，从而出现白屏现象。

### WKWebView的Cookie问题
Cookie 问题是目前 WKWebView 的一大短板，因为UIWebView的cookie和NSHTTPCookieStorage是共通的。所以说NSHTTPCookieStorage 的单例对象会自动将 cookie 添加到相应的 UIWebView 的 request 上。每次加载完页面以后 UIWebView 上生成的 cookie 也会自动保存到 NSHTTPCookieStorage 单例对象。

WKWebView 的 cookie 处理起来就很麻烦了。为了将 NSHTTPCookieStorage 单例对象的 cookie 设置到 WKWebView，需要在 loadRequest 的时候将 cookie 设置到 request 的 http 头上。代码如下：

```swift
- (void)loadRequest:(NSURLRequest *)req {
    NSMutableURLRequest *request = req.mutableCopy;
    NSString *urlString = request.URL.absoluteString;
    if (urlString && [urlString rangeOfString:@"www.xxx.com"].location != NSNotFound) {
        NSHTTPCookie *cookie = // specific cookie to be set;
        NSArray* cookies = [NSArray arrayWithObjects: cookie, nil];
        NSDictionary * headers = [NSHTTPCookie requestHeaderFieldsWithCookies:cookies];
        [request setAllHTTPHeaderFields:headers];
    }
    [(WKWebView *)self.wkWebView loadRequest:request];
}
```

如果有 AJAX 的 request 的话需要设置脚本（原理就是将本地 cookie 通过 js 设置到页面的 document.cookie）：

```swift
WKUserContentController* userContentController = WKUserContentController.new;
WKUserScript * cookieScript = [[WKUserScript alloc] 
    initWithSource: @"document.cookie = 'TeskCookieKey1=TeskCookieValue1';document.cookie = 'TeskCookieKey2=TeskCookieValue2';"
    injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:NO];
// again, use stringWithFormat: in the above line to inject your values programmatically
[userContentController addUserScript:cookieScript];
WKWebViewConfiguration* webViewConfig = WKWebViewConfiguration.new;
webViewConfig.userContentController = userContentController;
WKWebView * webView = [[WKWebView alloc] initWithFrame:CGRectMake(/*set your values*/) configuration:webViewConfig];

```

由于采用了新的实现机制，WKWebView 获取的 cookie 不会被设置到 NSHTTPCookieStorage 单例对象上。如果想要获取 WKWebView 上面的 cookie，同样可以是用 document.cookie。但是这种方式无法获取到 httpOnly 的 cookie。为了获取到所有的 WKWebView 上的 cookie。可以使用 NSURLProtocol，在 NSURLProtocol 得到调用的时候去取所有的相关 cookie。但是 WKWebView 默认并不走 NSURLProtocol，需要使用自定义protocol（私有API)：

```swift
Class cls = NSClassFromString(@"WKBrowsingContextController");
    SEL sel = NSSelectorFromString(@"registerSchemeForCustomProtocol:");
    if ([(id)cls respondsToSelector:sel]) {
        // 把 http 和 https 请求交给 NSURLProtocol 处理
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
        [(id)cls performSelector:sel withObject:@"http"];
        [(id)cls performSelector:sel withObject:@"https"];
#pragma clang diagnostic pop
    }

    [NSURLProtocol registerClass:[CustomURLProtocol class]];
```
所以，各位看官可以看出，目前最大的问题就是**WKWebView 发起的请求不会自动带上存储于 NSHTTPCookieStorage 容器中的 Cookie。** 这就很尴尬了啊。比如我有一个这样的cookie：
>name=yy;value=yy;domain=yy.qq.com;expires=Sat, 02 May 2019 23:38:25 GMT；

通过 UIWebView 发起请求，则会自动带上，但是WKWebView则不会。。。

不过这有牵扯到另一个问题了：

### WKWebView NSURLProtocol问题

这个方法不仅危险，而且也被腾讯的同学证明了有2个致命的缺陷：post 请求 body 数据被清空和对ATS支持不足（这一点我觉得目前应该不紧迫）

* post 请求 body 数据被清空
由于 WKWebView 在独立进程里执行网络请求。一旦注册 http(s) scheme 后，网络请求将从 Network Process 发送到 App Process，这样 NSURLProtocol 才能拦截网络请求。在 webkit2 的设计里使用 MessageQueue 进行进程之间的通信，Network Process 会将请求 encode 成一个 Message,然后通过 IPC 发送给 App Process。出于性能的原因，encode 的时候 HTTPBody 和 HTTPBodyStream 这两个字段被丢弃掉了(参考苹果源码： 
https://github.com/WebKit/webkit/blob/fe39539b83d28751e86077b173abd5b7872ce3f9/Source/WebKit2/Shared/mac/WebCoreArgumentCodersMac.mm#L61-L88 及bug report: https://bugs.webkit.org/show_bug.cgi?id=138169)。
**因此，如果通过 registerSchemeForCustomProtocol 注册了 http(s) scheme, 那么由 WKWebView 发起的所有 http(s)请求都会通过 IPC 传给主进程 NSURLProtocol 处理，导致 post 请求 body 被清空；**

**这个问题在`loadRequest`的过程中，一样会有问题**
一种解决思路是：
中间做个映射，简单来说就是自己生成自定义的schema，将post的body先copy到header中（不会丢失），然后注册custom schema，用`NSUrlProtocol`拦截请求，进行重组装.


## WKWebView的使用
WKWebVIew是UIWebView的代替品，新的WebKit框架把原来的功能拆分成许多小类。本例中主要用到了WKNavigationDelegate,WKUIDelegate,WKScriptMessageHandler三个委托和配置类WKWebViewConfiguration去实现webView的request控制，界面控制，js交互，alert重写等功能。 使用WKWebView需要引入 `#import <WebKit/WebKit.h>`

### js中alert的拦截
在WKWebview中，js的alert是不会出现任何内容的，你必须重写WKUIDelegate委托的`runJavaScriptAlertPanelWithMessage message`方法，自己处理alert。类似的还有Confirm和prompt也和alert类似。

### app调js方法
没什么区别

### Config
WKWebView的config需要在初始化之前设置好

### WKWebView的相关的代理方法
WKWebView的相关的代理方法分别在WKNavigationDelegate和WKUIDelegate以及WKScriptMessageHandler这个与JavaScript交互相关的代理方法。

* WKNavigationDelegate: 此代理方法中除了原有的UIWebView的四个代理方法，还增加了其他的一些方法。
* WKUIDelegate: 此代理方法在使用中最好实现，否则遇到网页alert的时候，如果此代理方法没有实现，则不会出现弹框提示。
* WKScriptMessageHandler: 此代理方法就是和JavaScript交互相关，具体介绍参考下面的专门讲解。

### WKWebView下面添加自定义View
>因为我们有个需求是在网页下面在添加一个View，用来展示此链接内容的相关评论。在使用UIWebView的时候，做法非常简单粗暴，在UIWebView的ScrollView后面添加一个自定义View，然后根据View的高度，在改变一下scrollView的contentSize属性。以为WKWebView也可以这样简单粗暴的去搞一下，结果却并不是这样。
>首先改变WKWebView的scrollView的contentSize属性，系统会在下一次帧率刷新的时候，再给你改变回原有的，这样这条路就行不通了。我马上想到了另一个办法，改变scrollView的contentInset这个系统倒不会在变化回原来的，自以为完事大吉。后来过了两天，发现有些页面的部分区域的点击事件无法响应，百思不得其解，最后想到可能是设置的contentInset对其有了影响，事实上正是如此。查来查去，最后找到了一个解决办法是，就是当页面加载完成时，在网页下面拼一个空白的div，高度就是你添加的View的高度，让网页多出一个空白区域，自定义的View就添加在这个空白的区域上面。这样就完美解决了此问题。具体可参考Demo所写，核心代码如下:

感谢[Time_for](https://www.jianshu.com/p/9513d101e582)同学的实践，本人打算去写一个demo验证下，稍后更新。

## 参考文件
* http://blog.csdn.net/Tencent_Bugly/article/details/54668721
* https://danleechina.github.io/things-to-know-about-WKWebView-UIWebView/
* http://liuyanwei.jumppo.com/2015/10/17/ios-webView.html


