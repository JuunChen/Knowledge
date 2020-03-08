### 1. RunLoop 的作用有哪些？为什么要有 RunLoop ?

1. 保持程序的持续运行。没有 RunLoop 的程序运行完毕就会立即退出。

   UIApplicationMain 函数中就启动了一个 RunLoop ，因此该函数一直没有返回。

   这个默认启动的 RunLoop 是和主线程相关联的。

2. 处理 App 中的各种事件

3. 节省 CPU 的资源：该做事的时候做事，不该做事的时候休息

   

   RunLoop 其实也是一个对象，我们如果能拿到这个对象就可以做很多底层的操作。

   iOS 中有两种方式可以拿到这个对象。

### 2. RunLoop 与线程的关系？

1. 每条线程都有



一个3秒的timer，在scrollView滑动的时候不响应，为什么？

runloop只能运行在一种模式下，timer默认是被加到default模式下。当scrollview滑动时runloop会进入 UITrackingMode，但是这个模式下没有这个timer，所以这个timer就不会响应。

解决办法：把timer同时加到defalutMode和UITrackingMode中，或者把它加到CommomMode中都行。





