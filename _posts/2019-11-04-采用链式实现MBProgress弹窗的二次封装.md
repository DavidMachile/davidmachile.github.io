---
layout:     post
title:      采用链式实现MBProgress弹窗的二次封装
subtitle:   技术成长之路
date:       2019-11-04
author:    DM
header-img: img/post-bg-cook.jpg
catalog: true
tags: iOS 链式 MBProgress
---

**简述**

​		在开发阶段，我们时常需要给用户展示提示语。因近期项目需要整改，趁此机会，做了基于MBProgress弹窗的二次封装。此次封装同时采用链式结构来重新修改了MBProgress的调用方式。

​		在封装之前需要提到一个链式编程思想。链式编程思想，我最初是从masonry框架中了解到的。这种思想不同于面向对象和面向过程，它的真正含义是将多个操作（多行代码）通过点号(.)链接在一起成为一句代码,使代码可读性好。例如a(1).b(2).c(3)

- 链式编程特点：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值）

**封装功能**

​		1.新增操作成功、失败和警告上图下文样式

​		2.支持上中下三个位置展示设置

​		3.新增自定义模式和普通样式展示

**链式封装**

​		目前提供以下方法

```objective-c


/**
*设置显示模式
*/
- (DMProgressManager *(^)(DMProgressHUDMode))hudMode;


/**
*显示状态(失败，成功，警告)
*/
- (DMProgressManager *(^)(DMProgressHUDStateType))hudState;

//- (DMProgressManager *(^)(DMProgressHUDPosition))position;

/**
*显示成功并自动消失
*/
+ (void)DM_showHUDWithSuccess:(NSString *)showString;

/**
*显示错误并自动消失
*/
+ (void)DM_showHUDWithError:(NSString *)showString;

/**
*显示警告并自动消失
*/
+ (void)DM_showHUDWithWarning:(NSString *)showString;

/**
*显示纯文字并自动消失
*/
+ (void)DM_showHUDWithText:(NSString *)showString;


/**
*显示状态自定义（自动消失）
*/
+ (void)DM_showHUDWithState:(void (^)(DMProgressManager *make))block;

/**
*直接消失
*/
+ (void)dissmissHUDDirect;

/**
.showMessage(@"需要显示的文字")
*/
- (DMProgressManager *(^)(NSString *))message;


/**
*一张图时的图片名字
*/
- (DMProgressManager *(^)(NSString *))imageStr;

```



**链式调用**

```objective-c
if (indexPath.row == 0) {
[DMProgressManager DM_showHUDWithText:@"最长不超14个字"];
}
if (indexPath.row == 1) {
[DMProgressManager DM_showHUDWithText:@"最长不超过14个字，如果超出14个字换行"];
}
if (indexPath.row == 2) {
[DMProgressManager DM_showHUDWithSuccess:@"成功提示"];

}
if (indexPath.row == 3) {
[DMProgressManager DM_showHUDWithError:@"错误提示"];
}
if (indexPath.row == 4) {
[DMProgressManager DM_showHUDWithWarning:@"警告提示"];

}if (indexPath.row == 5) {
[DMProgressManager DM_showHUDWithState:^(DMProgressManager * _Nonnull make) {
make.message(@"错误提示");
make.hudState(DMProgressHUDStateTypeError);
}];

}if (indexPath.row == 6) {
[DMProgressManager DM_showHUDWithState:^(DMProgressManager * _Nonnull make) {
make.message(@"最长不超14个字");
}];

} if (indexPath.row == 7) {
[DMProgressManager DM_showHUDWithText:@"据刘向《说苑·善说》记载：春秋时代，楚王母弟鄂君子皙在河中游玩，钟鼓齐鸣。摇船者是位越人，趁乐声刚停，便抱双桨用越语唱了一支歌。鄂君子皙听不懂，叫人翻译成楚语。就是上面的歌谣。歌中唱出了越人对子皙的那种深沉真挚的爱恋之情，歌词声义双关，委婉动听。是中国最早的译诗，也是古代楚越文化交融的结晶和见证。"];
}
```





