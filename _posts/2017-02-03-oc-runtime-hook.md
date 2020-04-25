---
layout: post
title:  "Objective-c Method Swizzling引发的死循环"
author: "Zongquan"
comments: true
---

在OC中：

API： class_addMethod往一个Class里添加method

API： class_getInstanceMethod或class_getClassMethod可以判断某个SEL是否存在于Class

API： method_exchangeImplementations 交换方法。 

最近工作上做了一件事，简单点说就是需要把一些特定Class里的方法func，替换成Hook_func，当Hook_func执行完之后，再执行func。于是很简单地想到了往Class添加一个Hook_func，然后再交换func与Hook_func，就能到达目的。 

但是，在实现后，却出现了死循环，纠其原因，因为部分Class间存在着继承关系，没有正确地将Method添加到正确的Class中导致。 

当时为了解决这个问题，重新去理清楚了一下Class中 SEL 与 Method的关系。SEL是一个选择器，相当于指向一个Method的指针，将SEL指向不同的Method，它就会有不一样的特性，method_exchangeImplementations也就是交换SEL指向的Method值来实现方法交换的。 

在调用class_getInstanceMethod时，是会检查superClass的。 

如下图，当ClassB继承ClassA时, ClassB中没有SEL func，而ClassA中有SEL func。

![](/assets/images/oc_method_swizzling1.png)

这时调用class_getInstanceMethod(ClassB, @selector(func))，是能拿到SEL func的Method的，然后再将ClassB中添加SEL Hook_func后，变成下图。

![](/assets/images/oc_method_swizzling2.png)

对ClassB调用method_exchangeImplementations后，得到下图。

![](/assets/images/oc_method_swizzling3.png)

这里可以看到，其实ClassA中的SEL：func已经是指向Method:Hook_func。在对ClassB的实例调用SEL:func时，能达到之前预订的效果，即先执行Hook_func后再执行func。

但在这个时候，如果再对ClassA做类似ClassB的处理，将得到下图：

![](/assets/images/oc_method_swizzling4.png)

这时SEL:func和SEL:Hook_func都指向了Method:Hook_func，于是便出现了死循环的问题。

最终的解决方法从上图已经可以很明显的看出来了，即在对ClassB做处理的时候，添加的SEL:Hook_func不应该添加到ClassB上，而应该添加到ClassA上，如下图，则问题得已解决。

![](/assets/images/oc_method_swizzling5.png)

执行下面的这个函数，找到正确的Class，然后再往Class里添加方法和交换方法，问题就解决了：

```objective-c
Class GetSelectorInsClass(Class hclass, SEL sel, Method sel_method) {
    Class super_class = [hclass superclass];
    if (!super_class) {
        return hclass;
    }
    Method method = class_getInstanceMethod(super_class, sel);
    if (method != sel_method) {
        return hclass;
    }
    return GetSelectorInsClass(super_class, sel, sel_method);
}
```

