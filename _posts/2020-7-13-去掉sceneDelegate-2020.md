---
layout:     post
title:      去掉SceneDelegate
subtitle:   iOS技术成长之路
date:       2020-07-13
author:     DM
header-img: img/post-bg-debug.png
catalog: true
tags: iOS   
---

Xcode11之后新创建的工程会多出两个文件SceneDelegate。那么我们如何让它变回之前的那样的工程呢。

一、将这两个文件删除。
会报错
```
There is no scene delegate set. A scene delegate class must be specified to use a main storyboard file.
```

二、将Info.plist -> Open As -> Source Code将划框的部分删除。

三、将AppDelegate.m中的UISceneSession lifecycle注释掉或者删掉。

```
#pragma mark - UISceneSession lifecycle
- (UISceneConfiguration *)application:(UIApplication *)application configurationForConnectingSceneSession:(UISceneSession *)connectingSceneSession options:(UISceneConnectionOptions *)options {
    // Called when a new scene session is being created.
    // Use this method to select a configuration to create the new scene with.
    return [[UISceneConfiguration alloc] initWithName:@"Default Configuration" sessionRole:connectingSceneSession.role];
}

- (void)application:(UIApplication *)application didDiscardSceneSessions:(NSSet<UISceneSession *> *)sceneSessions {
    // Called when the user discards a scene session.
    // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
    // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
}
```

四、在didFinishLaunchingWithOptions中加入UIWindow。

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.backgroundColor = [UIColor whiteColor];

    self.window.rootViewController = [[ViewController alloc] init];;
    
    [self.window makeKeyAndVisible];
    return YES;
}
```
转载自：
[去掉sceneDelegate](https://blog.csdn.net/weixin_43864837/java/article/details/104232482)
