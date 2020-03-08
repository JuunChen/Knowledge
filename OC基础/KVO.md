### KVO 的实现

利用runtime创建一个新的子类，让instance的isa指向这个新的类。

这个新的类重写了父类的setter方法，来实现属性改变的监听。

具体的调用顺序如下：

- willChageValueForKey:
- 调用父类的setter方法
- didChageValueForKey:
- 触发监听器的监听方法 `- (void)observeValueForKeyPath: ofObject: change: context:`

### 如何手动触发KVO？

手动调用`willChageValueForKey:`和`didChageValueForKey:`

### 直接修改成员变量会触发KVO吗？

不会

### 什么时候适合手动调用KVO？

直接修改成员变量不会触发KVO，为保证KVO机制的正常运行，可以手动调用KVO。

```objective-c
- (void)someMethod
{
	[self willChageValueForKey:@"var"];
	_ivar = xxx;
	[self didChageValueForKey:@"var"];
}
```

