---
layout:     post
title:      2020-7-13-去掉sceneDelegate-2020
subtitle:   iOS技术成长之路
date:       2020-07-13
author:     DM
header-img: img/post-bg-ioses.jpg
catalog:    true
tags:       AI iOS
---

Xcode11 之后新创建的工程会多出两个文件 SceneDelegate。那么我们如何让它变回之前的那样的工程呢。

一、将这两个文件删除。
会报错

```
There is no scene delegate set. A scene delegate class must be specified to use a main storyboard file.
```

二、将 Info.plist -> Open As -> Source Code 将划框的部分删除。

三、将 AppDelegate.m 中的 UISceneSession lifecycle 注释掉或者删掉。

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

四、在 didFinishLaunchingWithOptions 中加入 UIWindow。

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
