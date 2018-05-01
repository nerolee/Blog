---
title: 在iOS9中突破微信App跳转的限制
date: 2016-01-26 23:21:01
tags: [iOS开发]
---

#### 前言

微信的普及程度相信不需要多言了，稍微回忆一下自己上一条短信是什么时候，上一条微信又是什么时候就知道了。
因此通过微信传播也是绝大部分App的一个重要手段，但是在iOS7微信出的新版本，就开始不允许从微信直接跳转到对应的App中了。
所谓上有政策，下有对策，各大App纷纷想出自己的解决方案，目前最最流行的就是在点击跳转到自己的App的时候出现一个蒙板，然后一个箭头指向右上角的更多按钮，然后用户从Safari打开当前网页，然后点击跳转。
这个版本算是一种无奈的妥协吧，毕竟用户用起来还是有点蛋疼的。
最近博主的一个Boss发现网易新闻竟然能直接从微信里打开自己的App，于是乎~这个“艰巨”的任务就落到了我的头上。中间研究的过程就不再赘述，隆重请出使用到的技术：iOS9 Universal Links！
这里不禁要夸一夸网易的iOS开发同学们，看文档看得够仔细啊！
附上Apple的官方文档，What’s new in iOS
不知大家能否在Search那一栏中找到Universal links:


> Universal links let you replace custom URL schemes with standard HTTP or HTTPS links. Universal links work for all users: If users have your app installed, the link takes them directly into your app; if they don’t have your app installed, the link opens your website in Safari. To learn more about universal links, see Support Universal Links.

P.S.该技术目前只能应用在iOS9中。

#### 开搞

OK，我们开搞。

1. 首先，参照上面提到的文档，创建一个叫apple-app-site-association的json文件(请不要加上.json或者其他任何后缀名)，文件的大致内容如下:

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            }
        ]
    }
}
```
下面来解释一下这个json文件里的字段：

**apps:** 目前只需要提供一个空数组就可以了，但是这个字段必须提供，不可不填。
**appID:**appID的格式大致是这样的：TeamID.App Bundle ID,以文档中提供的例子来说，Apple出的WWDC的应用的开发者Team的Team ID为：9JA89QQLNQ，这个应用的BundleID为：com.apple.wwdc。
**paths:**希望Safari在访问哪个路径时，跳转到对应的App，可以使用*作为通配符。

2. 将准备好的apple-app-site-association上传到web网站的根目录下。

3. 配置App的Entitlements file：选中App的Target，在Capabilities中打开Associated Domains



切记关联的域名需要使用applinks:作为开头。

4. 在AppDelegate中实现对应的delegate，大致代码如下：

```
-(BOOL)application:(UIApplication *)application 
continueUserActivity:(NSUserActivity *)userActivity 
restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{
    if (userActivity.webpageURL != nil)
    {
        
    }
}
```

userActivity中的webpageURL就是对应的从Web端跳转而来的网址，其中可以带上各种参数来供App进行不通的判断和操作。

5. Web页面上的JS代码大致如下：

```
var open_app = document.getElementById('open_app');
btn_open.addEventListener('click', function() {
	open_app.src = 'https://www.domain.com/site/download?force=1';
	setTimeout(function() {
		location.href = 'https://www.domain.com/site/download?force=1';
	}, 1000);
}, false);
```

和Web端的同学一开始怎么也搞不定点击按钮跳转，找了好久终于发现为了提高手机端Web页面的点击响应速度，我们的Web端默认是使用touch事件来代替click的，但是在Universal Links的跳转中必须使用click。
另外，页面初始页和要跳转的页的域名必须是不同的，否则这个跳转事件也不会调起对应的App！另外，在进行Universal Links的调试时，建议先删除App，然后重新编译，运行。

#### 总结

稍稍总结一下，Universal Links的工作原理大体上就是，App第一次启动后，发现自己是支持Associated Domains的，就去Associated Domains中描述的域名的根目录去下载一个名叫apple-app-site-association的文件，之后系统运行的Safari，其他应用的SFSafariViewController, WKWebView, 或者UIWebView都会受到这个apple-app-site-association文件的影响，在用户点击那个跳转的链接时，系统就会启动对应的App。
不得不说，以后每个版本的What’s New还是必须得看得更加仔细一些，同时需要大大提高对于新技术的敏感度。
最后的最后，再次感慨一下腾讯确实强大，但是还是苹果爸爸更屌！
