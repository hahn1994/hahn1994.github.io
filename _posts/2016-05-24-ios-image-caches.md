---
layout:   post
title:    iOS网络图片缓存
date:     2016 May 24 19:22:23
category: iOS
tags:     网络加载 图片缓存机制 URL iOS
---

## 一、加载网络图片

### 1.简单方法展示网络图片

{% highlight objc %}
// picsURL是一个存储URL地址的数组
// choice是选择图片的索引数
UIImage *image=[UIImage imageWithData:
	[NSDatadataWithContentsOfURL:[NSURLURLWithString:[picsURL objectAtIndex:choice]]]];
[self.imageView setImage:image];
{% endhighlight %}

- 看似获取网络上的图片十分简单，实际这样的操作有很大的性能问题：
	- 这种方法是同步获取的，如果图片十分大的话，界面就会卡死了
    
### 2.采取异步获取的方法来展示图片

{% highlight objc %}
//_data是一个NSMutableData
- (void)connection:(NSURLConnection*)connection didReceiveResponse:(NSURLResponse*)response{
//可以在显示图片前先用本地的一个loading.gif来占位。
   UIImage *img = [[UIImage alloc] initWithContentsOfFile:@"loading.gif"];
   [self.imageView setImage:img];
   _data = [[NSMutableDataalloc] init];
   //保存接收到的响应对象，以便响应完毕后的状态。
   _response = response;
}
- (void)connection:(NSURLConnection*)connection didReceiveData:(NSData*)data {
//_data为NSMutableData类型的私有属性，用于保存从网络上接收到的数据。
//也可以从此委托中获取到图片加载的进度。
   [_data appendData:data];
   NSLog(@"%lld%%", data.length/_response.expectedContentLength * 100);
}
- (void)connection:(NSURLConnection*)connection didFailWithError:(NSError*)error{
   //请求异常，在此可以进行出错后的操作，如给UIImageView设置一张默认的图片等。
}
- (void)connectionDidFinishLoading:(NSURLConnection*)connection{
   //加载成功，在此的加载成功并不代表图片加载成功，需要判断HTTP返回状态。
   NSHTTPURLResponse*response=(NSHTTPURLResponse*)_response;
   if(response.statusCode == 200){
        //请求成功
        UIImage *img=[UIImage imageWithData:_data];
        [self.imageView setImage:img];
   }
}
{% endhighlight %}

*****

## 二、处理网络图片缓存步骤

在开发移动应用的时候，因为手机流量、网速、内存等这些因素，当我们的移动应用是针对互联网，并要频繁访问网络的话，对网络优化这块就显得尤为重要了。比如某个应用要经常显示网络图片，就不能每次显示图片都去网络上下载，那太耗费时间也太耗费流量，这时就要对网络图片进行缓存了。
{: .notice}

1. 根据图片URL查找内存是否有这张图片，有则返回图片，没有则进入第二步
2. 查找物理存储是否有这张图片，有则返回图片，没有则进入第三步
3. 从网络上下载该图片，下载完后保存到内存和物理存储上，并返回该图片
    - **注**：因为URL包含特殊字符和长度不确定，要对URL进行MD5处理或其他处理

### 下面是针对以上步骤的代码讲解：
- 内存缓存图片处理
    - 使用NSMutableDictionary存储图片UIImage，数组的Key为该图片的URL地址

{% highlight objc %}
// 缓存图片到内存上
[memCache setObject:image forKey:key];  
{% endhighlight %}

- 物理缓存图片处理
    - 把图片保持到物理存储设备上，则直接使用NSFileManager，把URL作为文件名保存
- 网络图片下载处理
    - 图片使用异步下载，下载完后把图片保持到NSMutableDictionary和物理存储上

*****

## 三、SDWebImage类库的处理方式

- SDWebImages是一个在网络图片缓存处理上比较好的第三方类库
- 特点：
    - 依赖的库很少.功能全面。
    - 自动实现磁盘缓存:
    - 缓存图片名字是以MD5进行加密的后的名字进行命名.(因为加密那堆字串是唯一的)
    - [imageViewsd_setImageWithURL:v.fullImageURL placeholderImage:[UIImage imageNamed:@”xxxxx”]].
    - 就一个方法就实现了多线程\带缓冲等效果.(可用带参数的方法,具体可看头文件)
- 下面是一部分源码解析

### SDImageCache.h文件:

{% highlight objc %}
@interface SDImageCache : NSObject
{
    NSMutableDictionary *memCache;//内存缓存图片引用
    NSString *diskCachePath;//物理缓存路径
    NSOperationQueue *cacheInQueue, *cacheOutQueue;
}
+ (SDImageCache *)sharedImageCache;  
//保存图片  
- (void)storeImage:(UIImage *)image forKey:(NSString *)key;  
//保存图片，并选择是否保存到物理存储上  
- (void)storeImage:(UIImage *)image forKey:(NSString *)key toDisk:(BOOL)toDisk;  
//保存图片，可以选择把NSData数据保存到物理存储上  
- (void)storeImage:(UIImage *)image imageData:(NSData *)data forKey:(NSString *)key toDisk:(BOOL)toDisk;  
//通过key返回UIImage  
- (UIImage *)imageFromKey:(NSString *)key;  
//如果获取内存图片失败，是否可以在物理存储上查找  
- (UIImage *)imageFromKey:(NSString *)key fromDisk:(BOOL)fromDisk;  
- (void)queryDiskCacheForKey:(NSString *)key delegate:(id <SDImageCacheDelegate>)delegate userInfo:(NSDictionary *)info;  
//清除key索引的图片
- (void)removeImageForKey:(NSString *)key;  
//清除内存图片
- (void)clearMemory;
//清除物理缓存
- (void)clearDisk;
//清除过期物理缓存
- (void)cleanDisk;
@end    
{% endhighlight %}

### SDImageCache.m文件

{% highlight objc %}
@implementation SDImageCache  
#pragma mark NSObject  
- (id)init
{
    if ((self = [super init]))  
    {
        // Init the memory cache  
        memCache = [[NSMutableDictionary alloc] init];  

        // Init the disk cache  
        NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);  
        diskCachePath = [[[paths objectAtIndex:0] stringByAppendingPathComponent:@"ImageCache"] retain];  

        if (![[NSFileManager defaultManager] fileExistsAtPath:diskCachePath])  
        {  
            [[NSFileManager defaultManager] createDirectoryAtPath:diskCachePath  
                                      withIntermediateDirectories:YES  
                                                       attributes:nil  
                                                            error:NULL];  
        }
        // Init the operation queue  
        cacheInQueue = [[NSOperationQueue alloc] init];  
        cacheInQueue.maxConcurrentOperationCount = 1;  
        cacheOutQueue = [[NSOperationQueue alloc] init];  
        cacheOutQueue.maxConcurrentOperationCount = 1;  
#if TARGET_OS_IPHONE  
        // Subscribe to app events  
        [[NSNotificationCenter defaultCenter] addObserver:self  
                                                 selector:@selector(clearMemory)  
                                                     name:UIApplicationDidReceiveMemoryWarningNotification  
                                                   object:nil];  

        [[NSNotificationCenter defaultCenter] addObserver:self  
                                                 selector:@selector(cleanDisk)  
                                                     name:UIApplicationWillTerminateNotification  
                                                   object:nil];  
#if __IPHONE_OS_VERSION_MIN_REQUIRED >= __IPHONE_4_0  
        UIDevice *device = [UIDevice currentDevice];  
        if ([device respondsToSelector:@selector(isMultitaskingSupported)] && device.multitaskingSupported)  
        {  
            // When in background, clean memory in order to have less chance to be killed  
            [[NSNotificationCenter defaultCenter] addObserver:self  
                                                     selector:@selector(clearMemory)  
                                                         name:UIApplicationDidEnterBackgroundNotification  
                                                       object:nil];  
        }  
#endif  
#endif  
    }  
    return self;  
}  
- (void)dealloc  
{  
    [memCache release], memCache = nil;  
    [diskCachePath release], diskCachePath = nil;  
    [cacheInQueue release], cacheInQueue = nil;  
    [[NSNotificationCenter defaultCenter] removeObserver:self];  
    [super dealloc];  
}  
#pragma mark SDImageCache (class methods)  
+ (SDImageCache *)sharedImageCache  
{  
    if (instance == nil)  
    {  
        instance = [[SDImageCache alloc] init];  
    }  
    return instance;  
}  
#pragma mark SDImageCache (private)  
/*  
 *创建指定图片key的路径  
 */  
- (NSString *)cachePathForKey:(NSString *)key  
{  
    const char *str = [key UTF8String];  
    unsigned char r[CC_MD5_DIGEST_LENGTH];  
    CC_MD5(str, (CC_LONG)strlen(str), r);  
    NSString *filename = [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",  
                          r[0], r[1], r[2], r[3], r[4], r[5], r[6], r[7], r[8], r[9], r[10], r[11], r[12], r[13], r[14], r[15]];  
    return [diskCachePath stringByAppendingPathComponent:filename];  
}  
/*  
 *保存key和Data到物理存储  
 *keyAndData[0] ->key  
 *keyAndData[1] ->Data  
 */  
- (void)storeKeyWithDataToDisk:(NSArray *)keyAndData  
{  
    // Can't use defaultManager another thread  
    NSFileManager *fileManager = [[NSFileManager alloc] init];  
      
    NSString *key = [keyAndData objectAtIndex:0];  
    NSData *data = [keyAndData count] > 1 ? [keyAndData objectAtIndex:1] : nil;  
      
    //如果有数据，则保存到物理存储上  
    if (data)  
    {  
        [fileManager createFileAtPath:[self cachePathForKey:key] contents:data attributes:nil];  
    }  
    else  
    {  
        //如果没有data，则把UIImage转换为JPEG，并保存到物理存储上  
        // If no data representation given, convert the UIImage in JPEG and store it  
        // This trick is more CPU/memory intensive and doesn't preserve alpha channel  
        UIImage *image = [[self imageFromKey:key fromDisk:YES] retain]; // be thread safe with no lock  
        if (image)  
        {  
#if TARGET_OS_IPHONE  
            [fileManager createFileAtPath:[self cachePathForKey:key] contents:UIImageJPEGRepresentation(image, (CGFloat)1.0) attributes:nil];  
#else  
            NSArray*  representations  = [image representations];  
            NSData* jpegData = [NSBitmapImageRep representationOfImageRepsInArray: representations usingType: NSJPEGFileType properties:nil];  
            [fileManager createFileAtPath:[self cachePathForKey:key] contents:jpegData attributes:nil];  
#endif  
            [image release];  
        }  
    }  
    [fileManager release];  
} 
/*  
 *查找图片委托  
 */  
- (void)notifyDelegate:(NSDictionary *)arguments  
{  
    NSString *key = [arguments objectForKey:@"key"];  
    id <SDImageCacheDelegate> delegate = [arguments objectForKey:@"delegate"];  
    NSDictionary *info = [arguments objectForKey:@"userInfo"];  
    UIImage *image = [arguments objectForKey:@"image"];  
      
    if (image)  
    {  
        [memCache setObject:image forKey:key];  
          
        if ([delegate respondsToSelector:@selector(imageCache:didFindImage:forKey:userInfo:)])  
        {  
            [delegate imageCache:self didFindImage:image forKey:key userInfo:info];  
        }  
    }  
    else  
    {  
        if ([delegate respondsToSelector:@selector(imageCache:didNotFindImageForKey:userInfo:)])  
        {  
            [delegate imageCache:self didNotFindImageForKey:key userInfo:info];  
        }  
    }  
}  
/*  
 *查找物理缓存上的图片  
 */  
- (void)queryDiskCacheOperation:(NSDictionary *)arguments  
{  
    NSString *key = [arguments objectForKey:@"key"];  
    NSMutableDictionary *mutableArguments = [[arguments mutableCopy] autorelease];  
      
    UIImage *image = [[[UIImage alloc] initWithContentsOfFile:[self cachePathForKey:key]] autorelease];  
    if (image)  
    {  
#ifdef ENABLE_SDWEBIMAGE_DECODER  
        UIImage *decodedImage = [UIImage decodedImageWithImage:image];  
        if (decodedImage)  
        {  
            image = decodedImage;  
        }  
#endif  
        [mutableArguments setObject:image forKey:@"image"];  
    }  
      
    [self performSelectorOnMainThread:@selector(notifyDelegate:) withObject:mutableArguments waitUntilDone:NO];  
}  
#pragma mark ImageCache  
/*  
 *缓存图片  
 *  
 **/  
- (void)storeImage:(UIImage *)image imageData:(NSData *)data forKey:(NSString *)key toDisk:(BOOL)toDisk  
{  
    if (!image || !key)  
    {  
        return;  
    }  
      
    //缓存图片到内存上  
    [memCache setObject:image forKey:key];  
      
    //如果需要缓存到物理存储上，并data不为空，则把data缓存到物理存储上  
    if (toDisk)  
    {  
        if (!data) return;  
        NSArray *keyWithData;  
        if (data)  
        {  
            keyWithData = [NSArray arrayWithObjects:key, data, nil];  
        }  
        else  
        {  
            keyWithData = [NSArray arrayWithObjects:key, nil];  
        }  
        //后台线程缓存图片到物理存储上  
        [cacheInQueue addOperation:[[[NSInvocationOperation alloc] initWithTarget:self  
                                                                         selector:@selector(storeKeyWithDataToDisk:)  
                                                                           object:keyWithData] autorelease]];  
    }  
}  
/*  
 *保存图片到内存上，不保存到物理存储上  
 */  
- (void)storeImage:(UIImage *)image forKey:(NSString *)key  
{  
    [self storeImage:image imageData:nil forKey:key toDisk:YES];  
}  
/*  
 *保存图片到内存上，不保存到物理存储上  
 */  
- (void)storeImage:(UIImage *)image forKey:(NSString *)key toDisk:(BOOL)toDisk  
{  
    [self storeImage:image imageData:nil forKey:key toDisk:toDisk];  
}  
  
/*  
 *通过key返回指定图片  
 */  
- (UIImage *)imageFromKey:(NSString *)key  
{  
    return [self imageFromKey:key fromDisk:YES];  
}
/*  
 *返回一张图像  
 *key：图像的key  
 *fromDisk：如果内存中没有图片，是否在物理存储上查找  
 *return 返回查找到的图片，如果没有则返回nil  
 */  
- (UIImage *)imageFromKey:(NSString *)key fromDisk:(BOOL)fromDisk  
{  
    if (key == nil)  
    {  
        return nil;  
    }  
    UIImage *image = [memCache objectForKey:key];  

    if (!image && fromDisk) //如果内存没有图片，并且可以在物理存储上查找，则返回物理存储上的图片  
    {  
        image = [[[UIImage alloc] initWithContentsOfFile:[self cachePathForKey:key]] autorelease];  
        if (image)  
        {  
            [memCache setObject:image forKey:key];  
        }  
    }
    return image;  
}  
  
- (void)queryDiskCacheForKey:(NSString *)key delegate:(id <SDImageCacheDelegate>)delegate userInfo:(NSDictionary *)info  
{  
    if (!delegate)  
    {  
        return;  
    }
    if (!key)  
    {  
        if ([delegate respondsToSelector:@selector(imageCache:didNotFindImageForKey:userInfo:)])  
        {  
            [delegate imageCache:self didNotFindImageForKey:key userInfo:info];  
        }  
        return;  
    }  
      
    // First check the in-memory cache...  
    UIImage *image = [memCache objectForKey:key];  
    if (image)  
    {  
        // ...notify delegate immediately, no need to go async  
        if ([delegate respondsToSelector:@selector(imageCache:didFindImage:forKey:userInfo:)])  
        {  
            [delegate imageCache:self didFindImage:image forKey:key userInfo:info];  
        }  
        return;  
    }
    NSMutableDictionary *arguments = [NSMutableDictionary dictionaryWithCapacity:3];  
    [arguments setObject:key forKey:@"key"];  
    [arguments setObject:delegate forKey:@"delegate"];  
    if (info)  
    {  
        [arguments setObject:info forKey:@"userInfo"];  
    }  
    [cacheOutQueue addOperation:[[[NSInvocationOperation alloc] initWithTarget:self selector:@selector(queryDiskCacheOperation:) object:arguments] autorelease]];  
}
/*  
 *从内存和物理存储上移除指定图片  
 */  
- (void)removeImageForKey:(NSString *)key  
{  
    if (key == nil)  
    {  
        return;  
    }  
    [memCache removeObjectForKey:key];  
    [[NSFileManager defaultManager] removeItemAtPath:[self cachePathForKey:key] error:nil];  
}  
/*  
 *清除内存缓存区的图片  
 */  
- (void)clearMemory  
{  
    [cacheInQueue cancelAllOperations]; // won't be able to complete  
    [memCache removeAllObjects];  
}
/*  
 *清除物理存储上的图片  
 */  
- (void)clearDisk  
{  
    [cacheInQueue cancelAllOperations];  
    [[NSFileManager defaultManager] removeItemAtPath:diskCachePath error:nil];  
    [[NSFileManager defaultManager] createDirectoryAtPath:diskCachePath  
                              withIntermediateDirectories:YES  
                                               attributes:nil  
                                                    error:NULL];  
}
/*  
 *清除过期缓存的图片  
 */  
- (void)cleanDisk  
{  
    NSDate *expirationDate = [NSDate dateWithTimeIntervalSinceNow:-cacheMaxCacheAge];  
    NSDirectoryEnumerator *fileEnumerator = [[NSFileManager defaultManager] enumeratorAtPath:diskCachePath];  
    for (NSString *fileName in fileEnumerator)  
    {  
        NSString *filePath = [diskCachePath stringByAppendingPathComponent:fileName];  
        NSDictionary *attrs = [[NSFileManager defaultManager] attributesOfItemAtPath:filePath error:nil];  
        if ([[[attrs fileModificationDate] laterDate:expirationDate] isEqualToDate:expirationDate])  
        {  
            [[NSFileManager defaultManager] removeItemAtPath:filePath error:nil];  
        }  
    }  
}
@end
{% endhighlight %}

- SDWebImage中的一些参数：
    - SDWebImageRetryFailed = 1<< 0,   默认选项，失败后重试
    - SDWebImageLowPriority = 1<< 1,    使用低优先级
    - SDWebImageCacheMemoryOnly = 1<< 2,   仅仅使用内存缓存
    - SDWebImageProgressiveDownload = 1<< 3,   显示现在进度
    - SDWebImageRefreshCached = 1<< 4,    刷新缓存
    - SDWebImageContinueInBackground =1 << 5,   后台继续下载图像
    - SDWebImageHandleCookies = 1<< 6,    处理Cookie
    - SDWebImageAllowInvalidSSLCertificates= 1 << 7,    允许无效的SSL验证
    - SDWebImageHighPriority = 1<< 8,     高优先级
    - SDWebImageDelayPlaceholder = 1<< 9     延迟显示占位图片