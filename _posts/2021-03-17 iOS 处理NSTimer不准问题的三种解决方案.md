---

layout:     post
title:      iOS 处理NSTimer不准问题的三种解决方案
subtitle:   GCD替代NSTimer 
date:       2021-03-17
author:     DM
header-img: img/post-bg-debug.png
catalog: true
tags: iOS 

---


#### 在开发工作中，我们经常会用到NSTimer 来实现定时器功能。但用心留意的小伙伴儿应该注意到了一个问题：定时器受其他UI界面刷新或者手势问题会出现卡顿现象，这会导致定时器不准确。究其原因：就是线程等待导致的问题。这也是面试中的一个经典案例。

####  具体原因：NSTimer 的runloop类型是NSDefaultRunloopMode 主线程中， 界面的刷新也在主线程中，UIScrollview滑动的过程中是在NSTrackingRunloopMode中，当我们在手指滑动过程中，系统会将NSDefaultRunloopMode 更改为NSTrackingRunloopMode，所以这也是为什么会出现NSTimer短暂暂停的原因

### 目前的解决方案有三种：

#### 1.通过更改RunloopMode 避免主线程阻塞
因为要避免主线程阻塞 所以也就是要避免使用NSDefaultRunloopMode。
查看到runloopMode的类型，可以看到 NSCommonRunloopMode （伪模式）不常用，而且是作为一个通用存在的。所以我们只需将NSDefaultRunloopMode 改为NSCommonRunloopMode调用就可以了。

但有一点需要注意：如果有多个NSCommonRunloopMode 执行不同任务 还是会出现卡顿导致定时器不准确问题。

```
self.timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```

#### 2.将NSTimer放在子线程中，当需要刷新UI界面时 再在主线程中进行。

这个方法是类似于将NSTimer 任务独立出来，也就是将定时和刷新UI的工作任务分开来。

```
- (void)timerMethod2 {

NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];

[thread start];

}

 -(void)newThread

{

[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(showTime) userInfo:nil repeats:YES];

[[NSRunLoop currentRunLoop] run];

}

```
这个方法是可以实现但会增加代码量，而且有一定的内存泄漏风险。
因为定时器的取消我们在delloc中进行的。如果对象没有完全销毁 就会导致内存泄漏问题出现。
所以比较推崇接下来的第三种方案。
#### 3.gcd实现定时器
gcd完全不受runloop影响，也就是说不会收到手势和UI刷新影响。所以它比NSTimer 更加准确。
```
 int count = 0;

     // 获得队列
    dispatch_queue_t queue = dispatch_get_main_queue();

    // 创建一个定时器(dispatch_source_t本质还是个OC对象)
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);

    // 设置定时器的各种属性（几时开始任务，每隔多长时间执行一次）
    // GCD的时间参数，一般是纳秒（1秒 == 10的9次方纳秒）
    // 何时开始执行第一个任务
    // dispatch_time(DISPATCH_TIME_NOW, 1.0 * NSEC_PER_SEC) 比当前时间晚3秒
    dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
    uint64_t interval = (uint64_t)(1.0 * NSEC_PER_SEC);
    dispatch_source_set_timer(self.timer, start, interval, 0);

    // 设置回调
    dispatch_source_set_event_handler(self.timer, ^{
        NSLog(@"------------%@", [NSThread currentThread]);
        count++;

        if (count == 4) {
            // 取消定时器
            dispatch_cancel(self.timer);
            self.timer = nil;
        }
    });

    // 启动定时器
    dispatch_resume(self.timer);
    
```
