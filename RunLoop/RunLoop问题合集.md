#### 1. Runloop有哪些作用？为什么要有Runloop？

保持程序的的运行；处理App的各种事件；节省资源，有事做事，没事休眠。

#### 2. Runloop与线程的关系？

一一对应的关系。它们保存在一个全局的字典里面，key是线程，value是RunLoop。

#### 3. 为什么NSTimer有时候不好使？

- NSTimer默认是被加在defaultMode中的，Runloop切换模式的时候，它就会失效。
- 如果Runloop任务比较繁重的话，会对timer造成阻塞

#### 4. autoreleasepool在什么时候被释放？

- 系统创建的autoreleasepool会在休眠前调用autoreleasepoolpage的pop和push
- 自己创建的autoreleasepool会在出大括号的时候调用pop

#### 5.`PerformSelector:afterDelay:`注意事项

该方法会创建一个NSTimer并添加到当前线程的runloop中，如果当前线程没有runloop，则改方法不会触发。

#### 6. Runloop的observer

```objc
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

#### 7. 事件响应过程

Source1接收系统事件后在回调 __IOHIDEventSystemClientQueueCallback() 内触发的 Source0，Source0 再触发的 _UIApplicationHandleEventQueue()。

`_UIApplicationHandleEventQueue()` 会把 `IOHIDEvent` 处理并包装成 `UIEvent` 进行处理或分发，其中包括识别 `UIGesture`/处理屏幕旋转/发送给 `UIWindow` 等。通常事件比如 `UIButton 点击`、`touchesBegin/Move/End/Cancel` 事件都是在这个回调中完成的

#### 8. 页面渲染的过程

当我们调用 `[UIView setNeedsDisplay]` 时，这时会调用当前 `View.layer` 的 `[view.layer setNeedsDisplay]`方法。

这等于给当前的 `layer` 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 `Runloop` 即将休眠，也就是 `beforeWaiting` 时才会进行绘制工作。