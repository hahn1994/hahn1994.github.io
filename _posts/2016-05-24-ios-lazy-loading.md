---
layout:   post
title:    iOS开发懒加载
date:     2016 May 24 12:03:00
category: iOS
tags:     懒加载 iOS 延迟加载
---

## 基本介绍

- 懒加载——也称为延迟加载，即在需要的时候才加载（效率低，占用内存小）。所谓懒加载，写的是其getter方法。
    - **注意** 重写get方法时，先判断对象当前是否为空，为空的话再去实例化对象
- 说的通俗一点，就是在开发中，当程序中需要利用的资源时。在程序启动的时候不加载资源，只有在运行当需要一些资源时，再去加载这些资源。
- 我们知道iOS设备的内存有限，如果在程序在启动后就一次性加载将来会用到的所有资源，那么就有可能会耗尽iOS设备的内存。
- 这些资源例如大量数据，图片，音频等等，所以我们在使用懒加载的时候一定要注意先判断是否已经有了，如果没有那么再去进行实例化

## 使用懒加载的好处：

1. 不必将创建对象的代码全部写在viewDidLoad方法中，代码的可读性更强
2. 每个控件的getter方法中分别负责各自的实例化处理，代码彼此之间的独立性强，松耦合
3. 只有当真正需要资源时，再去加载，节省了内存资源。

## 正常加载代码示例

- 没用懒加载的时候，从plist获取数据，返回一个数组，需要写在viewDidLoad方法中获取

{% highlight objc %}
@interface ViewController ()
@property (nonatomic, strong) NSArray *shopData;
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    _shopData = [NSArray arrayWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"shop" ofType:@"plist"]];
}
@end
{% endhighlight %}

- 显而易见，当控制器被加载完成后就会加载当前的shopData，假如shopData是在某些事件被触发的时候才会被调用，没必要在控制器加载完就去获取plist文件，如果事件不被触发，代表着shopData永远不会被用到，这样在viewDidLoad中加载shopData就会十分多余，并且耗用内存

## 使用懒加载代码示例

{% highlight objc %}
- (void)viewDidLoad {
    [super viewDidLoad];
}
- (NSArray *)shopData
{
    if (!_shopData) {
        _shopData = [NSArray arrayWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"shop" ofType:@"plist"]];
    }
    return _shopData;
}
@end
{% endhighlight %}

- 当需要用到shopData的时候，就会调用[self shopData]的方法（即getter方法），此时系统会去调用getter方法，然后再getter方法中获取plist文件内容，然后返回使用（需要注意在getter方法里切勿使用self.shopData，因为self.shopData会调用getter方法，造成死循环）
