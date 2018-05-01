---
title: 关于Swift编译慢并没什么卵用的解决办法
date: 2016-08-19 23:25:07
tags: [Swift]
---

目前开发中的项目使用了Swift和OC混编的，在感慨Swift本身的强大的前提下，也不得不面对引入Swfit带来的问题，比如App包的变大，比如Xcode对于Swift的支持不如OC等等，但是，对我来讲，最坑爹的还是引入了Swfit之后编译速度慢成了狗。
如果大家还没有体会过Swift的编译速度能够慢到什么程度，请义无反顾地将下面的代码加入到工程中去:

```
let myCompany = [
    "employees": [
        "employee 1": ["attribute": "value"],
        "employee 2": ["attribute": "value"],
        "employee 3": ["attribute": "value"],
        "employee 4": ["attribute": "value"],
        "employee 5": ["attribute": "value"],
        "employee 6": ["attribute": "value"],
        "employee 7": ["attribute": "value"],
        "employee 8": ["attribute": "value"],
        "employee 9": ["attribute": "value"],
        "employee 10": ["attribute": "value"],
        "employee 11": ["attribute": "value"],
        "employee 12": ["attribute": "value"],
        "employee 13": ["attribute": "value"],
        "employee 14": ["attribute": "value"],
        "employee 15": ["attribute": "value"],
        "employee 16": ["attribute": "value"],
        "employee 17": ["attribute": "value"],
        "employee 18": ["attribute": "value"],
        "employee 19": ["attribute": "value"],
        "employee 20": ["attribute": "value"],
    ]
]
```

OK, 在项目里加上上面的代码之后，如果不出什么意外的话（相信我，应该不会出什么意外的，除非你的开发机是一台超级计算机）你就可以双手离开键盘，起身出去吃饭了，如果运气好的话，你的项目应该能够在你吃完饭之后成功编译，当然了，其实我根本就没有成功编译这段代码，所以他具体要花多长时间我并不知，可能是一顿饭的时间，也可能需要天荒地老，海枯石烂 。

OK，回归正题，我们来讲一下，如何来提高Swift的编译速度，在讲如何提高Swfit的编译速度之前，我们先来将一些全局的设置，当然了，只要是对于Debug而言的，首先，在Build Settings的Debug Information Format中，见Debug得格式改为DWARF，因为我们在Debug的过程中一般是不需要这些Crash Symbol的，另外，Build Settings的Build Active Architecture Only，将Debug改为Yes，也就是说只编译当前选中的设备的架构，否则的话，编译时编译器就会将所有支持的Architecture都编一次（一般来讲现在都是armv7和arm64）。
好了，讲过这些通用的设置之后，我们来看一下加入了Swift之后App编译的过程。

如果有CocoaPods的话，编译对应的.a或者framework
编译xib等资源文件
编译Swift文件
编译.m， .cpp等文件
通常情况下，耗时最长的就是编译swift文件了，而且在编译的时候不像是编译.m文件，Xcode连个进度都不给，所以给人的感觉就是卡在这步不动了。

至于为什么编swift文件会这么慢，这里就要借助编译提供的额外的信息了。
还是Build Settings中，在Other Swift Flags中加入-Xfrontend -debug-time-function-bodies然后就可以在编译输出的界面看到每个函数编译花费的时间了。
当然了，更加简单的方法就是加一个Xcode的插件BuildTimeAnalyzer-for-Xcode

之后就是根据编译的时间来找茬了，下面列几个特别毁三观的情况，大家自己看。

```
let tableHeight = CGFloat(80 * 2 + 50 + 52 + 35) //3.1ms
let tableHeight:CGFloat = 80 * 2 + 50 + 52 + 35 //1588ms
```

两行代码编译时间差了500多倍，三观尽毁

```
//写法一
contactInfoArray.appendContentsOf(mobileItems)
contactInfoArray.appendContentsOf(telItems)
contactInfoArray.appendContentsOf(faxItems)
//写法二
contactInfoArray.appendContentsOf(mobileItems + telItems + faxItems)
写法一比写法二在编译时快了2000多ms
```

```
//写法一
if let index = localMessageIDs.indexOf(kCCAssistantMessageIDCardClaim) {
    if index < messageShowncount {
        if let oneMessage = messageObjectArray[index] as? CCAssistantMessageType6 {
            if oneMessage.localMessageID == kCCAssistantMessageIDCardClaim {
                oneMessage.fetchUm03UpdateInfo()
                dispatch_async(dispatch_get_main_queue(), {
                    let indexStr = "\(index)"
                    NSNotificationCenter.defaultCenter().postNotificationName(CCMessageAssistantDidUpdateMessagesNotification, object: nil, userInfo: [kMessageUpdateTypeModify:indexStr])
                    
                })
                ISSwiftInfo("Signal")
                dispatch_semaphore_signal(IOSemaphore)
                return
            }
        }
    }
}
//写法二
if let index = localMessageIDs.indexOf(kCCAssistantMessageIDCardClaim) {
    if index < messageShowncount {
        if let oneMessage = messageObjectArray[index] as? CCAssistantMessageType6 {
            if oneMessage.localMessageID == kCCAssistantMessageIDCardClaim {
                oneMessage.fetchUm03UpdateInfo()
                dispatch_async(dispatch_get_main_queue(), {
                    NSNotificationCenter.defaultCenter().postNotificationName(CCMessageAssistantDidUpdateMessagesNotification, object: nil, userInfo: [kMessageUpdateTypeModify:"\(index)"])
                    
                })
                ISSwiftInfo("Signal")
                dispatch_semaphore_signal(IOSemaphore)
                return
            }
        }
    }
}
```

写法一比写法二编译时快600ms

另外，再介绍一个最大的坑，在查看函数编译时间的时候，会发现有些swift文件的getter方法或者closure会被编译几百次，成为编译速度慢的罪魁祸首，最后分析下来，原来都是lazy （Lazy关键字的解释）这个关键字搞得鬼，只要是被lazy修饰的变量，会在每个swfit文件被编译的时候都编译一次，简直就是坑爹。

好了，稍微总结一下，根本原则就是，如果希望编译速度更快，那么就在代码中更多地显示声明变量的类型啦，容器元素的类型啦等等。就是不要让编译器在编译的时候帮你去分析类型。总得来讲就是，写代码写得越爽，编译时间很可能就会变长。

最后，点一下题吧，为什么说没什么卵用的解决方法呢，首先，lazy这个关键字没法去啊，去了之后编译的时间是变短了，但是运行的效率也降低了。另外一个最最关键的原因，架不住文件多啊，每个文件几百ms，几千个文件一加就崩溃了，另外还得加上CocoaPods的编译，xib资源文件的编译，以及.m这些文件的编译。总的来讲，就是——并没什么卵子用！
