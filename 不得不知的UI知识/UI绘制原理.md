#### `[UIView setNeedsDisplay]`的流程

当我们调用`[UIView setNeedsDisplay]`后，实际上并没有立刻发生视图的绘制工作，而是在之后的某一时机才会进行当前UI视图的真正绘制工作。这是为什么呢？

具体的流程如下所示：

- 当我们调用`[UIView setNeedsDisplay]`时
- 系统会立即调用`[view.layer setNeedsDisplay]`，相当于是在当前layer上打了一个脏标记
- 在当前Runloop将要结束时，系统会调用`[CALayer display]`方法，才进入到当前视图真正的绘制流程当中
- `[CALayer display]`会判断`[layer.delegate respondsToSelector:@selector(displayLayer:)]`
  - 如果它的delegate不响应，则进入系统默认的绘制流程，然后结束
  - 如果它的delegate响应，那么实际上就为我们提供了异步绘制的入口，然后结束

#### 系统的绘制流程

- CALayer内部创建一个 `backing store`即`CGContextRef`，在`drawRect:`方法中可以拿到它
- layer会判断它是否有代理
  - 如果没有代理，会调用`[CALayer drawInContext:]`方法
  - 如果有代理，会调用`[layer.delegate drawLayer:inContext:]`方法，然后做当前视图的绘制工作，这一部分是发生在系统的内部当中的
    - 然后在一个合适的时机给予我们一个回调方法`[UIView drawRect:]`。`drawRect:`的默认实现是什么都不做，给我们开这个口子，就允许我们在系统绘制的基础之上再做一些其它的绘制工作
- 不论是哪两个分支，最终都会由CALayer将backing store上传给GPU。这里的backing store就可以理解为最终的位图
- 系统默认的绘制流程结束

#### 如何实现异步绘制

如果`[layer.delegate respondsToSelector:@selector(displayLayer:)]`，即layer的代理实现了`displayLayer:`方法，就可以进入到异步绘制的流程当中。在这个流程中，我们需要做以下事情：

- 代理在子线程中生成对应的bitmap
- 回到主线程中设置该bitmap作为layer.contents属性的值

#### 离屏渲染

**在屏渲染**

指GPU的的渲染操作在`当前屏幕缓冲区中`进行

**离屏渲染**

指GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

**触发离屏渲染的操作**

- cornerRadius与maskToBounds同时设置
- 设置layer的阴影
- 设置layer蒙版
- 光栅化

**为何要避免离屏渲染**

触发离屏渲染时，会增加GPU的工作量，很有可能导致CPU和GPU工作耗时加起来的总耗时超出了16.7ms，可能会导致UI的卡顿和掉帧。