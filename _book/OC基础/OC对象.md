### Instance对象

```objectivec
NSObject *obj = [[NSObject alloc] init];
```

alloc出来的对象就是instance对象。

它在内存中存储的信息有：

- isa指针
- 其它成员变量

### class对象

```objectivec
[obj class];
[NSObject class];
object_getClass(obj);
```

它们都是class对象，每个内在内存中有且只有一个class对象。

它在内存中存储的信息有：

- isa
- superclass指针
- 类的属性信息
- 类的对象法法信息
- 类的协议信息
- 类的成员变量信息
- ...

注意，使用`-class`和`+class`方法都只会返回该类的class对象，而不会返回该类的meta-class对象。

### meta-class 对象

```objectivec
Class metaClass = object_getClass([NSObject class]);
```

每个类在内存中有且只有一个meta-class对象。

它在内存中存储的信息有：

- isa
- superlass指针
- 类方法信息
- ...

### object_getClass(id obj)

如果向该方法中传入instance对象，则返回该类的class对象。

如果向该方法中传入class对象，则返回该类的meta-class对象。

实际上该方法返回的时候obj的isa指向的对象。