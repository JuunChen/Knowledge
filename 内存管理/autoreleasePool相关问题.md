### 自动释放池

它实际上是一个栈结构，是由AutoreleasePoolPage对象组成的。

如果有很多页的话，它们之间是以双向链表的形式连接的。

调用AutoreleasePoolPage的push方法时会入栈一个哨兵对象（为了简化边界条件而设置的一个不存储数据的对象）。

对象调用autorease时，会将该对象的地址入栈。

调用AutoreleasePoolPage的pop方法时会从最后一个入栈的对象开始，发送release消息并出栈，直到上一个哨兵对象出栈。

### 我们自己创建的自动释放池

```
@autoreleasepool {

}
```

会在进入大括号的时候调用AutoreleasePoolPage的push方法，出大括号的时调用pop方法。

### 系统创建的自动释放池

```
 @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
 }
```

由于UIApplicationMain方法中开启了runloop，所以系统创建的自动释放池并不会出大括号。

但是系统在主线程的runloop中注册了2个observer

第一个observer监听了kCFRunLoopEntry事件，会调用push

第二个observer

- 监听了kCFRunLoopBeforeWaiting事件，会先调用pop，然后调用push
- 监听了kCFRunLoopBeforeExit事件，会调用pop

### 问题：autorelease对象什么时候释放？

1. 如果它是加到我们自己创建的自动释放池，那么在出大括号的时候会向它发送一次release消息。
2. 如果它是加到系统创建的自动释放池，那么在runloop即将休眠的时候会向它发送一次release消息。

### 问题：ARC下，方法中有局部对象，出方法后会立即释放吗？

释放的本质是引用计数为0。

如果编译器帮我们添加的代码是release，那么在发送release消息后它就会释放。

如果编译器帮我们添加的代码是autorelease，那么它是在runloop即将休眠前才能收到release消息。

### 问题：自动释放池的使用场景？

避免内存峰值过高：

```
NSArray *urls = <# An array of file URLs #>;
for (NSURL *url in urls) {
    @autoreleasepool {
        NSError *error;
        NSString *fileContents = [NSString stringWithContentsOfURL:url
                                        encoding:NSUTF8StringEncoding
                                         error:&error];
   	}
}
```

### 多层自动释放池嵌套的对象在哪一层释放

最里面的一层先 release。因为它先出大括号，先调用 AutoreleasePool 的 pop 方法。