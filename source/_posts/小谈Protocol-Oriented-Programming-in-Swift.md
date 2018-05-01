---
title: 小谈Protocol-Oriented Programming in Swift
date: 2016-01-21 22:32:10
tags: [iOS开发, Swift]
---

#### 前言

最近和老刘约好希望能够每周能够在这里留下点什么，往日的鸽王老刘上周竟然没有鸽！
反倒是我周末因为忘带电脑，导致无力一更，这周又天天加班到爆炸，竟然拖了大半周，趁着明天公司要开拔去开年会，今天早早溜了回来，想着不能再欠着了，今日就把这一更搞定吧~

小割一发

---

今天就就着WWDC的Protocol-Oriented Programming in Swift来小谈一记。
说到Protocol-Oriented Programming，就不得不提到与它对应的Class-Oriented Programming（又或者说Object-oriented programming），想到当年刚刚接触C++的时候，博主就对C++中的Class表示了赞叹，可以把一个个对象抽象出来，把一些方法封装起来，可以非常方便而且结构清晰地解决问题，重点是OOP，听起来就屌屌的啊，绝对的装逼一大利器。首先，我们来看看在传统OC上的Class都能够帮我做些什么

**Encapsulation：**封装显然不必多说，Class最最基本的属性就是封装了。
**Access Control：**这也不必多提，就像平时开车的时候踩了刹车，刹车灯就亮了一样，显然造汽车的时候给踩刹车这件行为加上了Setter方法，当刹车状态发生变化的时候，就顺便把刹车灯的状态也做了改变。
**Abstraction：**抽象自然也不需要多提，Class本身就是用来定义一件事物的抽象特点的。
**Namespace：**这点主要是用来解决命名冲突，例如ClassA可以定义一个叫sort的方法，ClassB也可以定义自己的sort方法。
**Expressive Syntax：**这点我偷偷地认为大约就是用Class来封装一些实体的对象，能够让后续代码的可维护性提高，更加便于使用人类可以理解的语法来表达一些行为。例如：定义了一个Person对象，那么这个对象的name，age等等属性就非常顺理成章了，同时eat，sleep这些方法在我们的语法中也就很正常了。
**Extensibility：**在OC中对于Class的扩展还是非常方便和实用了，可以subclass，可以category，也可以extension等等。
当然，以上提到的种种到了Swift中之后就不再是Class的特权了，在Swift中，Struct甚至是Enum都可以轻松拥有上述的能力。

于是乎，似乎在Swift中，Class留下的唯一优势就是Inheritance了，抽象好父类，然后子类再进行继承，父类可以定义一些通用的方法，如果子类不需要做特殊的事情，直接就可以不用管，如果子类要做的事情和父类不一样，那么将这些不一样的点单独再抽象出来，子类override就可以了，这样解决起问题来，简明而且优雅，简直不能更赞！

但是继承在给我们带来便利的同时，也不是完全没有开销的，我们先撇开那些因为水平不够而做出不合理的封装和继承不谈，哪怕是经过良好思考和设计的继承也不得不面对一些问题，其中一个问题便是Implicit Sharing，在日常实践中，在一个Class中保存一些数据必然是无法避免的，同时，如果做了继承，就很难避免子类本身是不需要完全继承父类的所有数据这个问题，往往都是子类中包含了很多本身完全用不到，但是却又不得不从父类继承而来的数据指针，OC为了性能的考虑对于这部分共享的数据是通过Implicit Sharing也就是隐式共享来实现的，也就是说子类本身不会再去拥有一份这些数据，而是通过隐式共享来共用父类的，但是一旦出现了共享，就势必会出现竞争，一旦出现竞争，为了保证数据的安全，就不得不再引入lock，一旦引入了lock，又势必会降低性能，甚至出现是死锁的情况。以至于Apple在他的官方开发文档中都做出了如下的声明

> It is not safe to modify a mutable collection while enumerating through it. Some enumerators may currently allow enumeration of a collection that is modified, but this behavior is not guaranteed to be supported in the future

另外，继承还有另外的一个问题，那就是一个儿子只能有一个老子——子类不可以从多个父类继承。同时，子类与父类的继承关系只能在子类被设计的时候就定义好，也就是儿子在出生的时候就确定了自己的唯一亲身父亲，是不可以再以后的成长过程中再去认个老子当自己的亲身老爹！那么，子类在后续开发过程中无论怎么扩展，都无法改变继承关系，不能引入更多抽象封装，除非对父类进行修改，但是一旦我们没有父类的源代码，那就只能蒙逼了！

再然后，在子类中Override父类的的一些方法的时候也必须小心翼翼，否则非常容易导致父类的业务逻辑链发生断裂，导致整个业务无法正常完成，甚至于，在子类中很难去判断，哪些方法是可以重写的，哪些方法是不能重写的。

还有一点，在Swift中，因为继承带来的多态会导致编译器类型检查出现Error，例如：

```
class Orderd {
    func precedes(other: Orderd) -> Bool { fatalError("implement me!")}
}

class Number : Orderd {
    var value: Double = 0
    override func precedes(other: Orderd) -> Bool {
        return value < other.value
    }
}
```

其中， return value < other.value编译器就一定会报错，为了解决这个Error，我们只能将代码改成

```
class Orderd {
    func precedes(other: Orderd) -> Bool { fatalError("implement me!")}
}

class Number : Orderd {
    var value: Double = 0
    override func precedes(other: Orderd) -> Bool {
        return value < (other as! Number).value
    }
}
```

但是显然这种强转是不安全的，因为谁也不知道Order又会被其他那个阿猫阿狗给继承，而那个类中又是否有value这个属性。 好了，给继承挑了这么多毛病，我们该谈谈Swift中的Protocol-Oriented Programming了。 首先，在Swift中，当你需要去抽象对象的时候，第一反映不应该想到Class，而是应该想到Protocol。对于上面提到的例子，如果我们使用Protocol来实现的话，就会变成这样：

```
protocol Orderd {
    func precedes(other: Self) -> Bool
}

struct Number : Orderd {
    var value: Double = 0
    func precedes(other: Number) -> Bool {
        return value < other.value
    }
}
```

而这个Protocol就会变成这样：

```
func binarySearch(sortedKeys:[T], forKey k: T) -> Int {
    var lo = 0
    var hi = sortedKeys.count
    while hi > lo {
        let mid  = lo + (hi - lo) / 2
        if sortedKeys[mid].precedes(k) { lo = mid + 1 }
        else {hi = mid}
    }
    return lo
}
```

#### 后记

好吧，其中本文主要都在介绍继承的side effect，并没有太多提到Protocol-Oriented Programming的优点，不过，Protocol-Oriented Programming没有上面提到的那些缺点应该就可以算是它的优点了吧^_^，下面附上WWDC2015年的视频链接，有兴趣的可以自己再去看看。
[https://developer.apple.com/videos/play/wwdc2015-408/](https://developer.apple.com/videos/play/wwdc2015-408/)

#### 最后的最后

在研究OOP的缺点的时候，看到了一个专业术语，叫做[内存墙](https://baike.baidu.com/item/内存墙)，有兴趣的同学可以自己点链接去看。下面附上一张图片来说明内存的进步速度是远远比不上CPU的，而OOP的语言在最初设计的时候并没有考虑到这个问题，可能会导致在内存的使用上并不能达到机器的最大性能，而导致性能有所削减，具体的就不再展开，有兴趣的可以去知乎上看看这个哥们的帖子：[面向对象编程的弊端是什么？-Milo Yip的回答](https://www.zhihu.com/question/20275578/answer/27046327)