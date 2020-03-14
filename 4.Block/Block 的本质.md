### block 简介

通常我们会这样使用block

```objc
//最简洁形式的block
^{};
//声明block并调用它
^{}();
//一般来说，我们会将block存起来，然后再调用它
void (^aBlock)(void) = ^{};//aBlock是变量的名称
aBlock();
//定义一个block的类型
typedef void (^CJBlock)(void);//CJBlock是block的类型
//用block作为函数的参数
+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animation;
```

### block 的本质

block的本质是一个OC对象，是一个结构体。

**写在大括号中的代码块会被封装成一个函数，然后创建一个结构体，把函数地址赋值给结构体中的成员变量**。

**执行block的时候，会找到这个函数地址，然后去调用它。**

### 源码窥探

我们写下一段简单的block代码，并查看它转成的C++源码：

```objc
void (^block)(void) = ^{ NSLog(@"Hello,world"); };//第一行
block();//第二行

//查看C++源码
void (*block)(void) = &__main_block_impl_0(__main_block_func_0, &_main_block_desc_0_DATA);//定义block
block->FuncPtr(block);//执行block内部的代码 	
```

我们来分析一下这段源码：

第一行

- block是一个变量
- 它的类型是`void(*)(void)`，这是一个指针类型
- 它指向的是`__main_block_impl_0()`函数的返回值的地址

第二行

- 调用`block()`实际上就是调用了`block->FuncPtr(block)`，即找到了block结构体中的成员变量`FuncPtr`去调用
- 调用`FuncPtr`的时候给它传递了一个参数:`block`

这里我要提出来两个问题：

1. **block指针指向的是什么？**
2. **block的调用是怎样的？**

**先来看第一个问题**，只要搞明白`__main_block_impl_0()`的返回值是什么，就知道block指向哪里了。

继续查看源码。

```objc
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* desc;
  //构造函数 返回值是__main_block_impl_0结构体对象
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

//__block_impl
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
}
```

这段稍微复杂一点，需要耐心一点看下去。

- 首先可以看到，`struct __main_block_impl_0`，说明这儿有一个结构体类型，它的类型名称是`__main_block_impl_0`。

- 它的第一个成员变量是 `impl`，这是一个结构体类型`struct __block_impl`，`__block_impl`中有4个成员变量。

- 它的第二个成员变量是`desc`，是一个指针类型。
- 它还有一个构造函数`__main_block_impl_0`，与结构体类型的名称相同，调用构造函数的时候会创建这个结构体，并返回结构体的地址。

我们初始化block的代码是：

```objc
void (*block)(void) = &__main_block_impl_0(__main_block_func_0, &_main_block_desc_0_DATA);
```

还记得上面我们说过"只要搞明白`__main_block_impl_0()`的返回值是什么，就知道block指向哪里了"吗？到这里想必你已经能回答这个问题了。

`__main_block_impl_0()`就是`struct __main_block_impl_0`的构造函数，它的返回值是一个一个类型是`struct __main_block_impl_0`的结构体。

所以block指向的其实就是一个类型是`struct __main_block_impl_0`的结构体。

**现在来看一下第二个问题。**

我们创建block的代码`^{ NSLog(@"Hello,world"); }`被转换成了`&__main_block_impl_0(__main_block_func_0, &_main_block_desc_0_DATA)`。

我们已经知道了`__main_block_impl_0`就是一个函数，所以这里`__main_block_func_0`和`&_main_block_desc_0_DATA`都是它的参数。那我们写的代码` NSLog(@"Hello,world"); `到哪里去了呢？

其实，参数`__main_block_func_0`实际上是一个函数，这个函数中的代码就是我们写在block中的代码:

```objc
static void __main_block_func_0(struct __main_block_imp_0 *__cself) {
  NSLog(@"Hello,world");
}
```

我们在看一下结构体的构造函数：

```objc
__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
}
```

可以看出来`__main_block_impl_0(__main_block_func_0, &_main_block_desc_0_DATA)`中的参数`__main_block_func_0`实际上是被赋值给了`__main_block_impl_0.impl.FuncPtr`。

调用`block()`的时候实际上就是调用了`block->FuncPtr(block)`，找到了block中的`FuncPtr`，然后去调用它。

我们再把整体的流程串一遍：

- 创建block实际上就是创建了一个结构体；并把代码包装成一个函数，然后用结构体中的一个成员变量`FuncPtr`指向这个函数
- 调用block实际上就是找到了结构体中的`FuncPtr`，直接调用它

**再补充一点：**上面已经介绍了`__main_block_impl_0`中的第一个参数`*fp`。那么它的第二个参数`struct __main_block_desc_0 *desc`是什么呢？

```objc
static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
```

desc实际上就是指向`__main_block_desc_0_DATA`的指针。`__main_block_desc_0_DATA`的类型是`struct __main_block_desc_0`。

这种结构体里面有两个成员变量，第一个是 `reserved`，这是一个保留变量，被赋值为0。第二个是`Block_size`，是block结构体的大小。

看到这里想必你对block的底层已经不那么陌生了。下一节我们说一说block的变量捕获。


