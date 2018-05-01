---
title: UITableView长按显示菜单的实现
date: 2015-12-14 21:56:46
tags: [iOS开发]
---

作为一个“资深”iOS开发程序猿，当然不能只是用这么个地方来低吟浅唱，无病呻吟了。这篇文章就作为技术博客的开篇吧。

*P.S. 其实本文的内容毫无技术含量，权当用来年纪渐长，记忆渐衰的提醒了。*

我是“有技术”的分割线

---

这次要实现的东西非常简单，就是希望能在tableView的Cell上长按，然后出现菜单，然后点击进行相应的操作。
最直接的实现方式就是在Cell上加一个LongPress的Gesture，捕获到LongPress的手势之后用UIMenuController把菜单显示出来。如果你已经能想到这些，那么恭喜你，你的战斗力已经很强了，至少已经学会使用Gesture，在Cell上算出Menu需要显示的区域，使用delegate，block或者notification甚至更加高深的机制如AssociatedObject通知另一个对象。但是博主比较懒，以上说的这些都不想自己做，特别是加Gesture，算区域等等，于是才有了下面这些哔了狗的内容。

首先，我们发现UITableView提供三个非常牛逼的Delegate方法


```
optional public func tableView(tableView: UITableView, shouldShowMenuForRowAtIndexPath indexPath: NSIndexPath) -> Bool
optional public func tableView(tableView: UITableView, canPerformAction action: Selector, forRowAtIndexPath indexPath: NSIndexPath, withSender sender: AnyObject?) -> Bool
optional public func tableView(tableView: UITableView, performAction action: Selector, forRowAtIndexPath indexPath: NSIndexPath, withSender sender: AnyObject?)
```

#### 下面开始哔狗第一式

这三个delegate方法只要有一个方法你没有实现，那么tableView都会华丽地无视你的长按事件，比如你只实现了其中的两个，没有实现第三个，那么恭喜你，你实现的那两个哥们会因为你无视了他们第三个兄弟而永远都不再理你。

OK，我按照要求把这三兄弟都实现了，shouldShowMenuForRowAtIndexPath返回了true，canPerformAction也返回了true，启动程序，长按Cell，是不是可以看到Menu了，点击弹出的Menu，performAction那个delegate是不是也被调用了！欣喜若狂有没有？！恭喜你，你可以进入哔狗第二式了，为何？因为他喵的显示了好多我不要的action啊！咦？不是有canPerformAction么，在这个delegate里面判断一下不就OK了么，英雄所见略同啊！博主也是这么想的，因为我只需要delete：这个方法，所以我只有当action == “delete:”的时候才返回了true，其他的都返回了false，OK，启动程序，长按，然后……然后就没有然后了？！毛都没显示出来啊！我感觉一定是我的打开方式有什么问题，于是去google了许久，StackOverFlow了许久，找到了一坨看似很有道理，实际上毛用都没有的答案，比如：这个TableView不能放在UIViewController里的啊，一定要放在UITableViewController里呢；哎呀~不要用这种方式实现啦，自己去加LongPress喽等等等等，这也促使博主怀着澎湃的心情，写下这篇“技术”帖。

#### 哔狗第二式

打坐苦思冥想许久，博主尝试性地在cell中override了canPerformAction

```
override func canPerformAction(action: Selector, withSender sender: AnyObject?) -> Bool {
if action == "delete:" {
    return true
} else {
    return false
    }
}
```

启动程序，长按，只显示了一个Delete按钮，炫！点击，Duang！程序华丽Crash！unrecognized selector sent to instance！

#### 哔狗第三式

为何ViewController里的performAction delegate不起作用了！也是，canPerformAction是在Cell里实现的，ViewController能知道个毛，于是，只能在cell里再实现override func delete(sender: AnyObject?)，启动程序，长按，终于成了！

再割一条

---

其实上面的全是废话，你只需要看下面的内容哦，亲~

在VC里实现tableView的那三个delegate，一个都不能少哦~
在cell里override canPerformAction，在你需要的action中返回true
在cell里实现你对应的action，然后通知VC，哪个action被点击了
如果把所有内容全看完的，请不要骂我~

#### 后记

写完这篇毫无技术含量的“技术”帖，我还是觉得我打开方式可能不对，感觉应该不需要使用这种“非主流”的方式实现这个需求，今天先这么不知甚解一把吧，等后面空下来再去细究一下~（其实我知道估计不会再去细究了~）