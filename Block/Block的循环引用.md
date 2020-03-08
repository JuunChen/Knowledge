### 常见的循环引用情况1

```objc
@interface Person : NSObject 
@property (nonatomic, copy) MyBlock block;
@end
...
...
- (void)testMethod
{
	Person *person = [[Person alloc] init];
	person.age = 10;
	person.block = ^{
		NSLog(@"%@", person.age);
	};
}
```

分析上述代码的流程：

- Block访问了局部变量person，所以会对person进行捕获
- person是auto变量，所以这是栈上的Block
- block是强指针，ARC自动把Block拷贝到堆上
- 拷贝到堆上的时，Block对捕获的对象进行内存管理。这里对象是用`__strong`修饰的，所以Block对对象产生了强引用。
- person的成员变量`_block`也强引用着Block

所以这里产生了循环引用

### 常见的循环引用情况2

```objc
@interface ViewController : UIViewController 
@property (nonatomic, copy) MyBlock block;
@end
...
...
- (void)testMethod
{
	self.block = ^{
		NSLog(@"%@", self);
	};
}
```

分析逻辑同上，block访问了auto局部变量`self`，self强引用了block。

### 常见的循环引用情况3

```objc
@interface ViewController : UIViewController 
@property (nonatomic, copy) MyBlock block;
@property (nonatomic, assign) int a;
@end
...
...
- (void)testMethod
{
	self.block = ^{
		NSLog(@"%@", _a);
	};
}
```

这里要注意，虽然没有在block中写明`self`，但是`_a`其实就是访问了self的局部变量，其实就是`self->_a`，所以这种情形也会产生循环引用。

### 常见的循环引用情况4

```objc
@interface ViewController : UIViewController 
@property (nonatomic, copy) MyBlock block;
@property (nonatomic, assign) int a;
@end
...
...
- (void)testMethod
{
	self.block = ^{
		[super xxxxxx];
	};
}
```

这里block使用了super，super实际上会找到self的父类去查找方法。block还是访问了self。

### 解决循环引用 -- ARC

#### 使用`__weak`解决

拷贝到堆上时Block不会产生强引用。并且对象被销毁后，指针会自动置nil。

#### 使用`__unsafe_unretained`解决

拷贝到堆上时Block不会产生强引用。对象被销毁后，指针不会自动置nil。

#### 使用`__block`解决

```objc
@interface Person : NSObject 
@property (nonatomic, copy) MyBlock block;
@end
...
...
- (void)testMethod
{
	__block Person *person = [[Person alloc] init];
	person.age = 10;
	person.block = ^{
		NSLog(@"%@", person.age);
		person = nil;
	};

	//必须要调用block，才能断开block对person的强引用
	person.block();
}
```

这种方式的缺陷是一定要等到block执行完毕，才能解决循环引用。

### 解决循环引用 -- MRC

#### 使用`__unsafe_unretained`解决

#### 使用`__block`解决

```objc
@interface Person : NSObject 
@property (nonatomic, copy) MyBlock block;
@end
...
...
- (void)testMethod
{
	__block Person *person = [[Person alloc] init];
	person.age = 10;
	person.block = ^{
		NSLog(@"%@", person.age);
		//不需要置nil
	};
	[person release];
	//不须要调用block，才能断开block对person的强引用
}
```

MRC与ARC这里会有差别，MRC下`__block`对象拷贝到堆上去的时候，内部的指向原对象的指针不会对原对象产生强引用，这种解决方案正是利用了这一点。

而ARC下把`__block`拷贝到堆上去的时候，那个指针会根据情况产生强引用或者不产生强引用。但在block结束前把它置为nil，就可以消除强引用。

#### MRC环境不支持`__weak`

