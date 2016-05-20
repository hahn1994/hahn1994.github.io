---
layout:   post
title:    iOS图片加载方式的区别
date:     2016 May 20 17:27:14
category: iOS
tags:     UIImage iOS 图片加载
---

> 正确选择图片加载方式能够对内存优化起到很大的作用

## 常见的图片加载方式有下面三种

{% highlight objc %}
// 方法1  
// imageNamed:
UIImage *imag1 = [UIImage imageNamed:@"image.png"];  

// 方法2  
// imageWithContentsOfFile:pathForResource:ofType:
UIImage *image2 = [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image.png" ofType:nil]];  

// 方法3  
// dataWithContentsOfFile:pathForResource:ofType:
NSData *imageData = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image.png" ofType:nil]];  
UIImage *image3 = [UIImage imageWithData:imageData]; 
{% endhighlight %}

## 第一种方法:imageNamed:

- 为什么有两种方法完成同样的事情呢？imageNamed的优点在于可以缓存已经加载的图片。苹果的文档中有如下说法：

> This method looks in the system caches for an image object with the specified name and returns that object if it exists. If a matching image object is not already in the cache, this method loads the image data from the specified file, caches it, and then returns the resulting object.

- 这种方法会首先在系统缓存中根据指定的名字寻找图片:
    - 如果找到了就返回。
    - 如果没有在缓存中找到图片，该方法会从指定的文件中加载图片数据，并将其缓存起来，然后再把结果返回。
- 对于同一个图像，系统只会把它Cache到内存一次，这对于图像的重复利用是非常有优势的。例如：你需要在 一个TableView里重复加载同样一个图标，那么用imageNamed加载图像，系统会把那个图标Cache到内存，在Table里每次利用那个图像的时候，只会把图片指针指向同一块内存。这种情况使用imageNamed加载图像就会变得非常有效。
- 如果在程序中经常需要重用的图片，那么最好是选择imageNamed方法。这种方法可以节省出每次都从磁盘加载图片的时间。

## 第二种方法和第三种方法本质是一样的:imageWithContentsOfFile:和imageWithData:

- imageWithContentsOfFile方法只是简单的加载图片，并不会将图片缓存起来，图像会被系统以数据方式加载到程序。当你不需要重用该图像，或者你需要将图像以数据方式存储到数据库，又或者你要通过网络下载一个很大的图像时，可以使用这种方式。
- 如果加载一张很大的图片，并且只使用一次，那么就不需要缓存这个图片。这种情况imageWithContentsOfFile比较合适,系统不会浪费内存来缓存图片。


