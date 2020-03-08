#### 事件传递链与响应者链

- 改变button的点击范围 pointInside
- hitTest

[点击这里跳转](事件传递链与响应者链.md)

#### imageName 与 imageWithContentsOfFile 的区别

**`imageName:`**加载的image会被系统缓存，当用完这个image对象后，将其置为nil，此时该图片并没有从内存中删除，而是被系统缓存在了内存里。

**`imageWithContentsOfFile:`**生成的image对象是不会被系统缓存起来的，当使用完image对象后，将其设置为nil，此时图片的资源是可以释放的。

**结论：**使用icon这种图片时，最好使用`imageName:`初始化，这样缓存会让整个过程更快速。如果加载的图片资源较大，那么最好用`imageWithContentsOfFile`，优化系统内存。

**还有一点很重要：**UIImage在加载图片时有类似于来加载的机制，初始化UIImage对象后，图片数据并没有加载到内存，只有当image显示时，图片的数据才会被加载到内存中。

#### 一个view上有一个吸附左侧的aLabel，还有一个吸附右侧的bLabel，如何让bLabel优先完整展示

设置bLabel的压缩阻力系数大于aLabel的压缩阻力系数。

压缩阻力是指一个视图保护其内容完整性的能力。`Content Compression Resistance`

内容吸附是指一个视图保持它的尺寸与其内容尺寸相匹配的能力。`Content Hugging`

#### 如何高效地切圆角

UIView（不包括其子类）可以直接使用`view.layer.cornerRadius`设置圆角。

cornerRadius 可以修改layer的圆角，但是它不能修改layer.contens，UIImageView的image实际上就是imageView.layer.contents，cornerRadius能修改它layer的圆角，但是不能修改contents的圆角。

UILabel与UIImageView这样的控件如果要设置圆角需要这样做：

```objc
label.layer.cornerRadius = 6.f;   
label.layer.maskToBounds = YES;   

imageView.layer.cornerRadius = 6.f;
imageView.layer.maskToBounds = YES; //裁剪掉layer以外的内容
imageView.clipsToBounds = YES;//这行代码与上面一行是等效的，只不过是直接通过view来操作


```

这样做会触发离屏渲染，如果要切的圆角个数少的话，这样做几乎没有影响。

如果要切的圆角个数多的话，这样做会严重影响性能。

推荐使用截取图片的方式切圆角，性能较高。

#### bounds 与 frame 的区别

#### position 与 anchorPoint

position与anchorPoint是layer的两个属性。我们除了用frame，还可以用这两个属性来修改视图的位置。

**`position`:**

- 用来设置CALayer在父层中的位置。
- 以父层的左上角为原点 (0, 0)
- UIView的center其实就是它layer的position。

**`anchorPonit`:**

- 决定着CALayer身上的哪个点会在position属性所指的位置
- 以自己的左上角为原点(0, 0)
- 取值范围为0 ~ 1，默认值是(0.5, 0.5)

#### drawrect 与 layoutsubviews 调用时机

#### UIView 与 CALayer 的区别

在创建UIView对象时，UIView内部会自动创建一个图层即CALayer。

当UIView需要显示到屏幕上时，会调用drawRect:方法进行绘图，并且将所有内容绘制在自己的图层上，绘图完毕后，系统会将图层拷贝到屏幕上。于是就完成了UIView的显示。

UIView本身不具备显示的功能，它内部的Layer才具有显示的功能。

#### 图像显示的原理 UI卡顿掉帧的原因

#### UI刷新的原理

#### 异步绘制的原理

#### 什么是离屏渲染

#### 如何提升UITableView的性能

#### 

#### 

#### 

#### 

