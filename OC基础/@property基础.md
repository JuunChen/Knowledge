### 目录
#### [@property 预编译指令](#1)
#### [在 category 中使用 @property](#2)
#### [子类使用父类属性的几种情况](#3)
#### [@property 有以下修饰词](#4)
#### [如果属性不加入修饰词系统会使用](#5)
#### [atomic vs nonatomic](#6)
#### [readonly / readwrite](#7)
#### [strong / retain / assign / weak / unsafe_unretained / copy](#8)
#### [getter=\<name> setter=\<name>](#9)
#### [](#10)

#### <h3 id = 1>@property 预编译指令</h3>
使用 @property 声明属性，默认添加了代码 `@synthesize var = _var`。  
编译器会自动：

- 创建成员变量 _var
- 声明并实现 _var 的 setter 与 getter

比如在类 Person 中声明了属性 name:  

```
@interface Person()
    
@property(nonatomic, copy) NSString *name;

@end
``` 
然后查看 Xcode 转成的 c++ 代码就可以验证：

```
struct Person_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	NSString *_name;
};
static NSString * _I_Person_name(Person * self, SEL _cmd) { return (*(NSString **)((char *)self + OBJC_IVAR_$_Person$_name)); }
static void _I_Person_setName_(Person * self, SEL _cmd, NSString *name) { objc_setProperty (self, _cmd, __OFFSETOFIVAR__(struct Person, _name), (id)name, 0, 1); 
``` 

如果我们设置 `@dynamic var;`，编译器不会做上边提到的几件事。此时：  

- 如果在代码中使用 _var，会编译不通过
- 如果使用了 self.var，而没有编写 getter 方法，在运行时会因找不到 getter 方法崩溃。

#### <h3 id = 2> 在 category 中使用 @property <h3>
编译器**仅仅只会**对 setter 与 getter 方法进行声明，不会实现，也不会生成成员变量。

#### <h3 id = 3> 子类使用父类属性的几种情况 <h4>
1. 父类在 .m 中声明了属性 var。  
 
	此时，子类如果也声明了属性 var，实际上子类在编译时会生成新的 _var 变量，实现新的 setter 与 getter 方法。  
	
	如果子类想使用父类的 \_var 而不想产生新的 \_var 实例，只需要声明 @dynamic var 即可。

2. 父类在 .h 中声明了属性 var。

	此时子类可以直接使用父类中的属性 self.var。  
	
	如果子类重新声明属性 var，编译器实际上也不会去实现该属性。该属性还是会由父类来实现，编译器会产生警告。添加 @dynamic 可以取消警告。
	
3. 父类在 .m 中声明了属性 var，在 .h 中暴露了属性 var 并添加修饰词 readonly。

	如果我们想在子类中修改 var。有两种比较合适的办法：
	- kvc
	- 在子类中重新声明属性 var 即可。再添加 @dynamic 去掉警告。

#### <h3 id = 4> @property 有以下修饰词 <h4>
- 原子性：atomic/nonatomic
- 读写权限：readonly/readwrite
- 内存管理：assign、weak、strong、copy、unsafe_unretained
- 方法名：getter=<name>、setter=<name>

如果使用 @synthesize 让编译器自动生成代码，这些修饰词会影响这些生成的代码。

#### <h3 id = 5> 如果属性不加入修饰词系统会使用 <h3>
系统会默认使用 atomic、readwrite、assign。

#### <h3 id = 6> atomic vs nonatomic <h3>

#### <h3 id = 7> readonly / readwrite <h3>

#### <h3 id = 8> strong / retain / assign / weak / unsafe_unretained / copy  <h3>

#### <h3 id = 9> getter=\<name\> setter=\<name\> <h3>

#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>
#### <h3 id = 10> <h3>

	
	

