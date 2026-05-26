---
layout:     post
title:      iOS widget 组件开发
subtitle:   iOS技术成长之路
date:       2024-10-25
author:     DM
header-img: img/post-bg-ioses.jpg
catalog:    true
tags:       Widget iOS 视频
---

## 背景

为了增加客户端的用户体验，在 iOS14 里，苹果更新了扩展组件，引入了新的 UI 组件：WidgetKit
而舍弃了 iOS14 以下版本的 Today Extension 组件

//图片 1

现在很多应用都有小组件功能，可以让用户很快使用 app 的主导功能。
比如今日头条、腾讯视频、优酷视频、抖音等都有这方面的功能。

//图片 2
苹果官方文档这样介绍 widget：
Today 视图中的应用扩展称为小部件。小部件使用户可以快速访问当前重要的信息。例如，用户打开今天视图以检查当前股票价格或天气状况，查看今天的日程安排，或执行快速任务，例如将项目标记为完成。用户倾向于经常打开“今日”视图，他们希望他们感兴趣的信息能够立即可用。

如果用户启用了今天小部件可以出现在 iOS 设备的锁定屏幕上。他们通过转到“设置”>“Touch ID 和密码”>“通知视图”在“锁定时允许访问”区域执行此操作。

[官网传送门](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Today.html)

## 特点

- 1.确保内容始终是最新的

- 2.适当地响应用户交互

- 3.表现良好（特别是 iOS 小部件必须明智地使用内存，否则系统可能会终止它们）

由于用户与 Today 小部件的交互快速且有限，因此您应该设计一个简单、流线型的 UI，突出显示用户感兴趣的信息。一般来说，限制小部件中交互项目的数量是个好主意。特别要注意 iOS 小部件不支持键盘输入。

## 使用 Xcode Today 模板

Xcode Today 模板为主体类（名为 TodayViewController）、Info.plist 文件和接口文件（即故事板或 xib 文件）提供了默认的头文件和实现文件。

默认情况下，Today 模板提供以下 Info.plist 键和值（此处针对 OS X 目标显示）：

```object-c
<key>NSExtension</key>
    <dict>
        <key>NSExtensionPointIdentifier</key>
        <string>com.apple.widget-extension</string>
        <key>NSExtensionPrincipalClass</key>
        <string>TodayViewController\</string>
    </dict>
```

如果您使用自定义视图控制器子类，请使用自定义类名替换键的 TodayViewController 值 NSExtensionPrincipalClass。

IOS。如果您不想使用模板提供的情节提要文件，请删除 NSExtensionMainStoryboard 键并添加 NSExtensionPrincipalClass 键，使用视图控制器的名称作为值。

您创建 Today 小部件所做的大部分工作都涉及设计 UI 和实现执行自定义功能的视图控制器子类。

## 设计用户界面

为获得最佳效果，请使用自动布局来设计 Today 小部件的视图。

因为 Today 视图中的空间是有限的，并且预期的用户体验是快速而集中的，所以默认情况下您不应该创建太大的小部件。在这两个平台上，Today 小部件必须适合 Today 视图的宽度，但它可以增加高度以显示更多内容。

使用 Xcode Today 模板创建的 Today 小部件包括标准边距插图的自动布局约束。要获取计算的插入值，请实现该 widgetMarginInsetsForProposedMarginInsets:方法。（模板的主要视图控制器符合 NCWidgetProviding 提供此方法的协议。）确保在这些标准边距插图内绘制所有小部件内容。要了解有关设计小部件外观的更多信息，请参阅 iOS 人机界面指南中的今日小部件。

如果小部件有额外的内容要显示，您可以依靠自动布局约束来适当地调整小部件的高度。如果您不使用自动布局，则可以使用该 UIViewController 属性 preferredContentSize 为小部件请求高度。例如：

```
- (void)receivedAdditionalContent {
  self.preferredContentSize = [self sizeNeededToShowAdditionalContent];
}

```

## 笔记

不要为您的小部件指定一个需要用户滚动才能查看其所有内容的高度。

IOS。如果您想为内容的显示设置动画以与调整大小动画一致，请实现

```
// 将您的动画添加到 coordinator 参数中。

viewWillTransitionToSize:withTransitionCoordinator:usinganimateAlongsideTransition:completion:
```

要确保您的小部件获得适合在“今日”视图中显示项目的活力效果，请使用 notificationCenterVibrancyEffect.

## 实现

\##图 3

Include Configuration Intent 支持用户自定义配置就勾选，不支持不勾选。

### 1.结构分析

placeholder：占位图 （例如网络请求失败、发生未知错误、第一次展示小组件都会展示这个 view）

getSnapshot：快照
（为了在小部件库中显示小部件，WidgetKit 要求提供者提供预览快照，在组件的添加页面可以看到效果）

getTimeline：时间线
（在这个方法内可以进行网络请求，拿到的数据保存在对应的 entry 中，调用 completion 之后会到刷新小组件）

- 参数 policy：刷新的时机
  .never：不刷新
  .atEnd：Timeline 中最后一个 Entry 显示完毕之后自动刷新。Timeline 方法会重新调用
  .after(date)：到达某个特定时间后自动刷新
- !!!Widget 刷新的时间由系统统一决定，如果需要强制刷新 Widget，可以在 App 中使用 WidgetCenter 来重新加载所有时间线：WidgetCenter.shared.reloadAllTimelines()

Timeline 的刷新策略是会延迟的，并不一定根据你设定的时间精确刷新。同时官方说明了每个 widget 窗口小部件每天接收的刷新都会有数量限制

### 2.入口

```
    @main
    struct widgetTest: Widget {
        let kind: String = "widgetTest"

        var body: some WidgetConfiguration {
            StaticConfiguration(kind: kind, provider: Provider()) { entry in
                widgetTestEntryView(entry: entry)
            }
            .configurationDisplayName("My Widget")
            .description("This is an example widget.")
        }
    }
```

kind App 分辨 Widget 的标志。用户自定义的字符串，切记要保持唯一。宿主应用刷新某个 widget 时会用到
provider 确定 Widget 刷新的时间轴对象。他会告诉系统 Widget 刷新的时间和内容，但是注意，这里系统会自己做刷新优化，但结果往往不是我们所希望的。
Content Widget 的内容。其实就是 Widget 视图。上图绿色框
configurationDisplayName Widget 的名字。 上图红色框
description Widget 的介绍。上图棕色框
supportedFamilies Widget 支持的样式，支持的样式类型我们后续会讲。上图为小型样式
