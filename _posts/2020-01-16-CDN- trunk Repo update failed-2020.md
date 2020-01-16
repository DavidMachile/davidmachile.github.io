---
layout:     post
title:      CDN: trunk Repo update failed  
subtitle:   ios 技术成长之路
date:        2020-01-16
author:    DM
header-img: img/post-bg-debug.png
catalog: true
tags: iOS 
---


最近在新项目添加cocopods 时 ，pod update时出现了CDN: trunk Repo update failed 错误。

![](http://q3gnsl2e8.bkt.clouddn.com/picGO/20200116111503.png)

我首先想到的是，可能是pod 版本太低了。所以
``` pod --version
``` 查看了下pod 版本
1.8.3 最新版本 那就不是pod 版本太低导致的。

<p style="font-color:red font-size:16">然后查看了podfile文件之后发现是数据源没添加上。</p>
所以在podfile的最上面添加了
```
source 'https://github.com/CocoaPods/Specs.git'
```

添加之后保存podfile文件
然后终端重新运行 ```pod install``` 命令行即可。