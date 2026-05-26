---
layout:     post
title:      简介
subtitle:   iOS技术成长之路
date:       2025-04-09
author:     DM
header-img: img/post-bg-ioses.jpg
catalog:    true
tags:       SDWebImage iOS
---

SDWebImage 提供的简洁的获取远程 URL 图片的 API；平时开发中使用最多场景就是列表中的 cell 中要显示远程图片的需求，在具体的实现中要避免加载图片造成的界面卡顿，列表卡顿等现象的出现；所以需要编码实现如下功能：

- 使用占位图片显示 UI 界面，异步线程加载图片成功后刷新控件
- 缓存机制，下载过的图片做内存缓存和磁盘缓存
- app 内存吃紧的状态下移除缓存的内容

SDWebImage 的框架结构

![](https://upload-images.jianshu.io/upload_images/26731569-b2091a1adbae0f55.png)

SDWebImage 的框架结构

SDWebImage 的图片下载分类，只要一行代码就可以实现图片异步下载和缓存功能。

# 功能简介

1.  一个添加了 web 图片加载和缓存管理的 UIImageView 分类
2.  一个异步图片下载器
3.  一个异步的内存加磁盘综合存储图片并且自动处理过期图片
4.  支持动态 gif 图
    - 4.0 之前的动图效果并不是太好
    - 4.0 以后基于 FLAnimatedImage 加载动图
5.  支持 webP 格式的图片
6.  后台图片解压处理
7.  确保同样的图片 url 不会下载多次
8.  确保伪造的图片 url 不会重复尝试下载
9.  确保主线程不会阻塞

# 实现原理

1.  架构图（UML 类图）

![img](https://upload-images.jianshu.io/upload_images/26731569-6d7c81f0ae855ebf.png)

架构图（UML 类图）

1.  流程图（方法调用顺序图）

![img](https://upload-images.jianshu.io/upload_images/26731569-9442b23de0c7ca73.png)

# 目录结构

- Downloader
  ○ SDWebImageDownloader
  ○ SDWebImageDownloaderOperation
- Cache
  ○ SDImageCache
- Utils
  ○ SDWebImageManager
  ○ SDWebImageDecoder
  ○ SDWebImagePrefetcher
- Categories
  ○ UIView+WebCacheOperation
  ○ UIImageView+WebCache
  ○ UIImageView+HighlightedWebCache
  ○ UIButton+WebCache
  ○ MKAnnotationView+WebCache
  ○ NSData+ImageContentType
  ○ UIImage+GIF
  ○ UIImage+MultiFormat
  ○ UIImage+WebP
- Other
  ○ SDWebImageOperation（协议）
  ○ SDWebImageCompat（宏定义、常量、通用函数）

相关类名与功能描述
　　 SDWebImageDownloader：是专门用来下载图片和优化图片加载的，跟缓存没有关系
　　 SDWebImageDownloaderOperation：继承于 NSOperation，用来处理下载任务的
　　 SDImageCache：用来处理内存缓存和磁盘缓存（可选)的，其中磁盘缓存是异步进行的，因此不会阻塞主线程
　　 SDWebImageManager：作为 UIImageView+WebCache 背后的默默付出者，主要功能是将图片下载（SDWebImageDownloader）和图片缓存（SDImageCache）两个独立的功能组合起来
　　 SDWebImageDecoder：图片解码器，用于图片下载完成后进行解码
　　 SDWebImagePrefetcher：预下载图片，方便后续使用，图片下载的优先级低，其内部由 SDWebImageManager ：来处理图片下载和缓存
　　 UIView+WebCacheOperation：用来记录图片加载的 operation，方便需要时取消和移除图片加载的 operation
　　 UIImageView+WebCache：集成 SDWebImageManager 的图片下载和缓存功能到 UIImageView 的方法中，方便调用方的简单使用
　　 UIImageView+HighlightedWebCache：跟 UIImageView+WebCache 类似，也是包装了 SDWebImageManager，只不过是用于加载 highlighted 状态的图片
　　 UIButton+WebCache：跟 UIImageView+WebCache 类似，集成 SDWebImageManager 的图片下载和缓存功能到 UIButton 的方法中，方便调用方的简单使用
　　 MKAnnotationView+WebCache：跟 UIImageView+WebCache 类似
　　 NSData+ImageContentType：用于获取图片数据的格式（JPEG、PNG 等）
　　 UIImage+GIF：用于加载 GIF 动图
　　 UIImage+MultiFormat：根据不同格式的二进制数据转成 UIImage 对象
　　 UIImage+WebP 用于解码并加载 WebP 图片

# 工作流程

![img](https://upload-images.jianshu.io/upload_images/26731569-622f61bf33e2e8e9.png)

工作流程

- 入口 setImageWithURL\:placeholderImage\:options: 会先把 placeholderImage 显示，然后 SDWebImageManager 根据 URL 开始处理图片。
- 进入 SDWebImageManager-downloadWithURL\:delegate\:options\:userInfo:交给 SDImageCache 从缓存查找图片是否已经下载 queryDiskCacheForKey\:delegate\:userInfo:。
- 先从内存图片缓存查找是否有图片，如果内存中已经有图片缓存，SDImageCacheDelegate 回调 imageCache\:didFindImage\:forKey\:userInfo: 到 SDWebImageManager。
- SDWebImageManagerDelegate 回调 webImageManager\:didFinishWithImage: 到 UIImageView+WebCache 等前端展示图片。
- 如果内存缓存中没有，生成 NSInvocationOperation 添加到队列开始从硬盘查找图片是否已经缓存。
- 根据 URLKey 在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。
- 如果从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 imageCache\:didFindImage\:forKey\:userInfo:进而回调展示图片。
- 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 imageCache\:didNotFindImageForKey\:userInfo:。
- 共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片。
- 图片下载由 NSURLConnection(3.8.0 之后使用了 NSURLSession)，实现相关 delegate 来判断图片下载中、下载完成和下载失败。
- connection\:didReceiveData: 中利用 ImageIO 做了按图片下载进度加载效果。connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。
- 图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。
- 在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，imageDecoder\:didFinishDecodingImage\:userInfo: 回调给 SDWebImageDownloader。
- imageDownloader\:didFinishWithImage: 回调给 SDWebImageManager 告知图片下载完成。
- 通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片。
- 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程。
- SDImageCache 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。
- SDWebImagePrefetcher 可以预先下载图片，方便后续使用。

# 常见面试题

1.  图片文件缓存的时间有多长：1 周

<!---->

    _maxCacheAge ＝ kDefaultCacheMaxCacheAge

1.  SDWebImage 的内存缓存是用什么实现的？

<!---->

    NSCache

1.  SDWebImage 的最大并发数是多少？
    `maxConcurrentDownloads ＝ 6`

- 是程序固定死了，可以通过属性进行调整！

1.  SDWebImage 支持动图吗？GIF

```objectivec
1. #import <ImageIO/ImageIO.h>
2. [UIImage animatedImageWithImages:images duration:duration];
```

1.  SDWebImage 是如何区分不同格式的图像的
    - 根据图像数据第一个字节来判断的！
    - PNG：压缩比没有 JPG 高，但是无损压缩，解压缩性能高，苹果推荐的图像格式！
    - JPG：压缩比最高的一种图片格式，有损压缩！最多使用的场景，照相机！解压缩的性能不好！
    - GIF：序列桢动图，特点：只支持 256 种颜色！最流行的时候在 1998 ～ 1999，有专利的！

6.SDWebImage 缓存图片的名称是怎么确定的！

- `md5`
- 如果单纯使用 文件名保存，重名的几率很高！
- 使用 MD5 的散列函数！对完整的 URL 进行 md5，结果是一个 32 个字符长度的字符串！

  7.SDWebImage 的内存警告是如何处理的！

- - 利用通知中心观察
  - `- UIApplicationDidReceiveMemoryWarningNotification` 接收到内存警告的通知
  - 执行 `clearMemory` 方法，清理内存缓存！
  - `- UIApplicationWillTerminateNotification` 接收到应用程序将要终止通知
  - 执行 `cleanDisk` 方法，清理磁盘缓存！
  - `- UIApplicationDidEnterBackgroundNotification` 接收到应用程序进入后台通知
  - 执行 `backgroundCleanDisk` 方法，后台清理磁盘！
  - 通过以上通知监听，能够保证缓存文件的大小始终在控制范围之内！
  - `clearDisk` 清空磁盘缓存，将所有缓存目录中的文件，全部删除！
    实际工作，将缓存目录直接删除，再次创建一个同名空目录！

文章摘自<https://www.cnblogs.com/mysweetAngleBaby/p/15645139.html>
