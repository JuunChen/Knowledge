### 理论

我们之前的变量捕获一节中讨论的变量都是基本类型的。如果捕获的auto局部变量是对象类型的，那么还需要考虑到被捕获对象的内存管理，情况会变得复杂一些。为什么要强调是auto变量呢？因为如果是static变量的话，该变量会存储在DATA段，不会被释放，对它的内存管理没有意义，所以我们也无需考虑。

我们从一个实际的例子来入手分析这个问题（没有特殊说明，默认是在ARC环境下）：

```objc
typedef void (^MyBlock)(void);
- (void)testMethod
{
	NSObject *myObj = [[NSObject alloc] init];
	MyBlock block = ^{
		myObj;
	};
}
```

上述代码定义了一个auto的局部变量`myObj`，它的类型是`NSObject *`。查看转成的C++代码：

```objc
struct __main_block_impl_0 {
	struct __block_impl impl;
	struct __main_block_desc_0* desc;
	//构造函数 略过
	...
	//变量捕获
	NSObject *__strong myObj;
};
```

上述代码是block所指向的结构体，它其实和访问了基本类型auto变量的block的结构没有差异。

```objc
static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
  void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0}
```

上述代码是desc指向的结构体。如果和之前捕获基本类型auto变量的情况的对比，可以发现这里多出来了两个成员变量，都是指向函数的指针，分别是 **copy** 和**dispose**。

```objc
static void __main_block_copy_0(struct __main_block_impl_0 *dst, __main_block_impl_0 *src) {
  _Block_object_assign((void*)&dst->myObj, (void*)src->myObj, 3);
}

static void __main_block_dispose_0(struct __main_block_impl_0 *src) {
  _Block_object_dispose((void*)src->myObj, 3);
}
```

上述代码是copy和dispose分别指向的函数。

接下来直接阐述结论，**如果捕获到对象类型的auto变量，是怎样进行内存管理的**？

1. **栈上的block不会对捕获到的对象强引用**。我这样理解这样做的原因，虽然不是很严谨
   - 栈上的block随时都会被销毁，所以它保证不了自己的生存，它去强引用变量也没有意义。
2. **当block copy到堆上的时候**：
   - 会调用block内部的`_main_block_copy_0`函数
   - `__main_block_copy_0`函数会调用`_Block_object_assign`
   - `_Block_object_assign`会根据auto变量的修饰符（`__strong __weak __unsafe_unretain`）做出相应的操作，形成强引用或者弱引用。即对block的引用计数+1或者不+1。

3. **当block从堆中移除的时候**：
   - 会调用block内部的`__main_block_dispose_0`函数
   - `__main_block_dispose_0`函数会调用`_Block_object_dispose`
   - `_Block_object_dispose`函数会对auto变量进行内存管理，即对block的引用计数-1或者不-1

**简单点说：**

- 栈上的block不会对捕获到的对象强引用
- 如果block访问的对象是用`__strong`修饰的，堆上的block会对它强引用
- 如果block访问的对象是用`__weak`修饰的，堆上的block不会对它强引用

### 实际问题分析

**代码分析1：**

```objc
typedef void (^MyBlock)(void);
- (void)testMethod
{
    MyBlock myBlock;
  
    {
        NSObject *myObj = [[NSObject alloc] init];
        myBlock = ^{
          myObj;
        };
    }
  
  	NSLog(@"----%@", myObj);
}
```

​	这里的问题就是代码执行到`NSLog(@"----%@", myObj);`的时候，myObj对象被释放了吗？

​	答案是没有。我们梳理一下整个逻辑：

- 这里创建出来了一个代码块Block :`^{ obj; }`，它访问了局部变量，所以它会捕获变量。由于它捕获的是auto类型的局部变量，所以auto变量的类型与它一致，都是 `NSObject *`，并且用 `__strong`修饰。

  ```objc
  struct __main_block_impl_0 {
    	...
    	//变量捕获
      NSObject *__strong myObj;
  };
  ```

- myBlock 是一个强指针，所以ARC环境下编译器会自动对它进行copy。

- 由于它访问的变量是用`__strong`修饰的，所以copy调用到的`_Block_object_assign`函数会对myObjc的引用计数+1，产生强引用。

  

  虽然已经出了作用域，但是myObj被block强引用了，而block这个时候还没有被释放，所以myObj也没有被释放。

**代码分析2，以下代码，在MRC环境下：**

修改改一下代码，MRC下我们需要对obj手动进行内存管理，添加`[obj release]`。

```objc
- (void)testMethod
{
	MyBlock block;

	{
		NSObject *myObj = [[NSObject alloc] init];
		block = ^{
			myObj;
		};
		[myObj release];
	}

	NSLog(@"----%@", myObj);
}
```

这里会释放。

原因很简单，在MRC下，这是一个栈上的block，栈上的block不会对变量产生强引用。

如果我们对block添加了copy操作：

```objc
typedef void (^MyBlock)(void);
- (void)testMethod
{
	MyBlock block;

	{
		NSObject *myObj = [[NSObject alloc] init];
		block = [^{
			myObj;
		} copy];
		[myObj release];
	}

	NSLog(@"----%@", myObj);
}
```

​	这里则不会释放。

**代码分析3，使用`__weak的情况`：**

下边的代码，obj在什么时候被销毁？

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
  NSObject *obj = [[NSObject alloc] init];

  dispatch_after(dispatch_time(DISPATCH_TIME_NOW), (int64_t)(3.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    NSLog(@"---%@", obj);
  });
}
```

​	分析一下上边的代码：

- 点击屏幕

- 创建了auto类型的局部变量obj，并用 `__strong`修饰

- GCD 的参数 block访问到了obj，所以它是一个栈上的block

- 在ARC下，编译器会自动帮我们copy GCD函数中的参数block，于是它变成了堆上的block

- 由于obj的修饰符是`__strong`，所以在copy的时候，`_Block_object_assign`会把obj的引用计数+1

- `touchesBegan:withEvent:`代码执行完毕，已经出了作用域，但是obj还在被block强引用着，所以obj没有被释放

- 在3秒之后执行到了block，打印了obj。

- block销毁

- block在销毁的时候调用了`dispose`函数

- dispose函数对obj进行引用计数-1

- obj释放

  

  对照着上面的分析，也可以来梳理一下使用 `__weak`的情况，这里就不过多阐述了。

```objc
- (void)touchedBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
	NSObject *obj = [[NSObject alloc] init];

	__weak NSObject *weakObj = obj;
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW), (int64_t)(3.0 * NSEC_PER_SEC)), 				dispatch_get_main_queue(), ^{
		NSLog(@"---%@", weakObj);
	});
}
```

