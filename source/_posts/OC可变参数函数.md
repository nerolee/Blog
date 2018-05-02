---
title: OC可变参数函数
date: 2017-07-20 23:44:53
tags: [Objective-C]
---

今天实现一个功能的时候需要用到可变参数，突然发现这些平时不常用的小技巧自己掌握地还不是很好，基本上用到了都要去网上搜，与其去其他地方找，还不如自己把这些小技巧都收集起来，也方便自己以后使用。于是有了这个Tag的开篇。

话不多说，直接上代码

```
+(instancetype)tupleWithObjects:(id)object, ...NS_REQUIRES_NIL_TERMINATION
{
    Tuple *tuple = [[Tuple alloc] init];
    NSMutableArray *objects = [[NSMutableArray alloc] init];
    va_list argList;
    if(object != nil)
    {
        [objects addObject:object];
        va_start(argList, object);
        id arg;
        while((arg = va_arg(argList, id))) {
            [objects addObject:arg];
        }
        
        va_end(argList);
        
        [tuple.objects addObjectsFromArray:objects];
    }
    
    return tuple;
}
```

值得注意的是，在调用可变参数的函数时，记得要在最后一个参数后加上nil，否则在调用arg=va_arg(argList, id)时会发生Crash。


