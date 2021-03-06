# Masonry源码阅读
Masonry是一个轻量级的布局框架，基于AutoLayout进行封装。简单来说就是Masonry用更简洁的语法描述了一个视图相对另一个视图的位置。


## 点链式语法

Masonry中使用了大量的点链式语法，所以有必要在读源码前了解点链式语法的基本使用

![](images/Masonry/点链式语法片段1.png)

拿block里的一段代码 **make.left.equalTo(superview.left).offset(padding)** 分析下

1. **"."** 相当于调用getter方法，**.left**相当于相当于调用了left方法，left方法定义如下

![](images/Masonry/点链式语法片段2.png)

left方法有返回值，类型是 MASConstraint *

2. 接下来的**equalTo(superview.left)**，在OC语言里调用block会使用()，调用一般的方法使用的是[]，equalTo()定义如下

![](images/Masonry/点链式语法片段3.png)

可以看到equalTo()也是返回block，block有类型也是 MASConstraint * 的返回值

3. 接下来的**offset(padding)** 原理跟**equalTo(superview.left)** 是一样的

![](images/Masonry/点链式语法片段4.png)

> 从2，3两点可以看出，在每次调用完block后如果返回调用者对象本身，就可以实现的调用了。


## Masonry是怎么创建约束和怎么将约束添加到视图上的?

Masonry文件列表如下：

![](images/Masonry/源码文件.png)

从最基本的使用开始探究代码的流程

![](images/Masonry/点链式语法片段1.png)

- **makeConstraints**

![](images/Masonry/代码流程片段1.png)

在这里创建MASConstraintMaker * 对象，用来管理block里的像**NSLayoutAttributeLeft** 等这些属性，和创建约束。

- **make.left**

![](images/Masonry/点链式语法片段2.png)

继续往下查看

![](images/Masonry/代码流程片段2.png)

![](images/Masonry/代码流程片段3.png)

 1. 根据传进来的参数**layoutAttribute** 生成MASViewConstraint * 对象
 2. 如果链式语法像**make.left.right** 这样调用就会进入这里，生成复合型约束MASCompositeConstraint * 对象
 3. 链式调用的初始调用会进入这里，像**make.left**

> 总结来说**make.left** 做的事情是根据NSLayoutAttribute属性创建并返回MASViewConstraint * 对象

- **make.left.equalTo(superview.left)**

equalTo()的定义如下:

![](images/Masonry/点链式语法片段3.png)

继续往下查看

![](images/Masonry/代码流程片段4.png)

这里要做的事情就是给**layoutRelation** 和**secondViewAttribute** 这两个属性赋值，着重看一下**secondViewAttribute** 的赋值操作

![](images/Masonry/代码流程片段5.png)

 1. 如果**superview.left** 传入的是数字
 2. 如果**superview.left** 传入的是UIView类，就设置secondViewAttribute的layoutAttribute和firstViewAttribute的layoutAttribute一样
 3. 如果**superview.left** 传入的是MASViewAttribute类，直接赋值

> 总结一下，首先自动布局有一个公式**item1.attribute1 = multiplier × item2.attribute2 + constant** ，**make.left** 创建了一个**firstViewAttribute,firstViewAttribute** 的view属性，相当于封装了布局公式的左边；而**equalTo()** 则是将**secondViewAttribute** 赋值给**MASViewConstraint** 对象的**secondViewAttribute** 属性，并给**MASViewConstraint** 对象的**layoutRelation** 属性赋值，相当于封装了布局公式的右边

- **offset()**

![](images/Masonry/点链式语法片段4.png)

这里的逻辑也很简单，就是给**MASViewConstraint** 对象的**layoutConstant** 属性赋值

![](images/Masonry/代码流程片段6.png)

到这里**make.left.equalTo(superview.left).offset(padding)** 的代码流程就解读完了

- **[constraintMaker install]** 

 这个方法就是将我们前面编好的约束添加到视图上
 
![](images/Masonry/代码流程片段7.png) 

前面类似.left和.left.right都会将约束**MASConstraint** 对象加到一个数组里，执行install方法就会遍历数组里每个约束，将约束添加到视图上

继续往下查看

![](images/Masonry/代码流程片段8.png) 

![](images/Masonry/代码流程片段9.png) 

 1. 若**firstView** 和 **secondView** 有共同的父视图，则将约束添加到父视图上
 2. 若约束是size的约束(即width和height)，则将约束添加到本视图
 3. 否则将约束添加到本视图的父视图
 4. 最后把约束添加到视图上


> 整体上Masonry的代码设计并不复杂，框架工整，通过一层一层地继承，让每一层的子类各司其职，而且像对bug的跟踪和断言的使用，非常值得参考学习。
