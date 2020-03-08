### NSTimer

```
//第一种方式
NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
		NSLog(@"1");
}];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

//第二种方式
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0 repeats:YES block:^(NSTimer * _Nonnull timer) {
        
}];
```

使用 `timerWith...`的方法创建出来的timer需要手动添加到RunLoop中去。

使用`scheduledTimer...`的方法创建出来的timer已经被系统添加到当前线程的runloop中的defaultModel里面了。

### CADisplayLink

```
CADisplayLink displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(linkTest)];
[displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
[displayLink invalidate]; //需要手动停止，移出 runloop
```

CADisplayLink添加到runloop中后，每当屏幕显示的内容刷新后都会向target发送selector消息。保证了屏幕刷新的频率与发送消息的频率一致。

### GCD Timer

```
//只执行一次
double delayInSeconds = 2.0;    
dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);   
dispatch_after(popTime, dispatch_get_main_queue(), ^(void){ 
          //执行事件    
});
```

```
//可以执行多次

@property (nonatomic, strong) dispatch_source_t timer;

self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue());
dispatch_source_set_timer(self.timer, DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC, 0 * NSEC_PER_SEC);
dispatch_source_set_event_handler(self.timer, ^{
    NSLog(@"111");
});
dispatch_resume(self.timer);
```

GCD Timer 是基于系统内核的，与runloop无关。当timer被释放时，定时器会停止。

### 使用NSTimer和CADisplayLink存在的问题

一、时间精度可能不准确

因为NSTimer和CADisplayLink都是基于runloop的，当runloop任务繁重的时候，会延误任务的触发。

再者，若timer被添加到defaultMode中，当runloop切换到uitrackingMode的时候，timer不会被调用，导致timer失效。

二、存在循环引用的风险

使用target-selector的方式创建NSTimer和CADisplayLink实例，实例会对target强引用，若target也对实例强引用，则会导致循环引用。

解决办法有三种：

1. 使用block
2. 使用NSProxy做转发。target -----> timer，timer -----> proxy，proxy - - -> target。
3. 使用gcd timer。