# 包含Eigen对象作为成员的结构
---

## 总结
如果你在你的结构里定义了[固定大小可量化类型][1]的成员变量，那么你必须重载它的`new`操作符来保证指向对象的指针是位于16字节整数倍的位置上。好消息是，Eigen提供了一个宏`EIGEN_MAKE_ALIGNED_OPERATOR_NEW`来帮助你完成这个任务。

### 什么样的代码需要修改？
---
像下面这样的代码需要修改：
```cpp
class Foo
{
  ...
  Eigen::Vector2d v;
  ...
};
...
Foo *foo = new Foo;
```
文字描述就是：你的类的成员变量包含有[固定大小可量化Eigen对象][1]，同时你还在动态生成这个类的一个实例。

## 这种代码要如何修改？

非常简单，你只需要在你类的public域添加宏`EIGEN_MAKE_ALIGNED_OPERATOR_NEW`就可以了，就像下面这样：
```cpp
class Foo
{
  ...
  Eigen::Vector2d v;
  ...
public:
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW
};
...
Foo *foo = new Foo;
```
这个宏会让"new Foo"返回一个内存对齐的指针。
如果你非常讨厌使用这种方式，那么你可以参考[其他方式][othersolution]。

## 为什么必须这么做？
好的，假设你的代码是下面这样：
```cpp
class Foo
{
  ...
  Eigen::Vector2d v;
  ...
};
...
Foo *foo = new Foo;
```
一个[Eigen::Vector2d][vector2d]包含两个double数据，总共是128位。这个大小正好是一个SSE数据包的大小，针对这个vector的大多数操作都可以都可以使用SSE。但是SSE指令(至少Eigen使用的，速度最快的那一种)要求数据指针是128位对齐的。不然的话，程序将出现段错误。
因此，[Eigen]使用了下面两种手段自己处理了[Eigen::Vector2d][vector2d]的对齐任务：
  - 对于[Eigen::Vector2d][vector2d]的数组(两个double)，[Eigen]要求它们满足128位对齐。在GCC编译器下，这个是通过**attribute**((aligend(16)))实现的。
  - [Eigen]重载了[Eigen::Vector2d][vector2d]的new操作符来保证得到的指针满足128位对齐。

所以，你无需担心什么，[Eigen]已经帮你处理好了对齐任务...
...除了一种例外。如果你的代码是上面的Foo那种，然后，你像例子那样动态生成了一个对象。因为Foo类没有内存对齐的new操作符，返回的指向对象的指针foo并不能保证是128位对齐的。
属性对象v的对齐与否完全依赖与类对象的起始地址foo.如果指针foo不是对齐的，那么`foo->v`当然也不会是对齐的！
解决的办法是，用上一节讲的方法，给类Foo提供一个对齐的`new`操作符。

## 我因该将Eigen类型的成员都放在类定义的开头吗？

并没有这种强制要求。因为声明了128位对齐后[Eigen]会自己处理这个任务，如果有的成员在进行128位对齐的时候有这种需求，它们会根据类的具体情况自动调整分布的位置。代码像下面这样写就可以了：
```cpp
class Foo
{
  double x;
  Eigen::Vector2d v;
public:
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW
};
```

## 那么动态大小的矩阵和向量怎么处理？

像[Eigen::VectorXd]这样的动态矩阵和向量会自动分配自己成员的地址，所以，它们对齐的要求是自动完成的。它们并不会引起文章提到的问题。我们讨论的问题只会发生在[固定大小可量化的矩阵和向量][1]身上。

## 所以这其实是Eigen的一个bug？

不，这不是我们的bug。这更多的是从C++98语言特性中继承过来的问题，而且好像在后来的版本中已经得到修复了，详情请参看[这份文档](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2341.pdf)。

## 我能否有选择的执行对齐（根据模板参数）？

为了满足这种需求，我们提供了这样一个宏`EIGEN_MAKE_ALIGNED_OPERATOR_NEW_IF(NeedsToAlign)`。当`NeedToAlign`为真的时候，它跟`EIGEN_MAKE_ALIGNED_OPERATOR_NEW `的效果是一样的。如果`NeedToAlign`为假，它将使用默认分配器的对齐方式。
示例：
```cpp
template<int n> class Foo
{
  typedef Eigen::Matrix<float,n,1> Vector;
  enum { NeedsToAlign = (sizeof(Vector)%16)==0 };
  ...
  Vector v;
  ...
public:
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW_IF(NeedsToAlign)
};
...
Foo<4> *foo4 = new Foo<4>; // foo4 is guaranteed to be 128bit-aligned
Foo<3> *foo3 = new Foo<3>; // foo3 has only the system default alignment guarantee
```

## 其他问题

需要在每个类里面添加宏`EIGEN_MAKE_ALIGNED_OPERATOR_NEW`确实太麻烦了，所以你还有至少两种其他解决方案。

### 禁用对齐
---
第一种方法是禁用固定大小成员的对齐要求：
```cpp
class Foo
{
  ...
  Eigen::Matrix<double,2,1,Eigen::DontAlign> v;
  ...
};
```
这会导致使用v的时候无法使用量化。如果Foo里有函数多次访问这个成员，有可能v会被拷贝到一个对齐的临时vector中然后自动开启量化:
```cpp
void Foo::bar()
{
  Eigen::Vector2d av(v);
  // use av instead of v
  ...
  // if av changed, then do:
  v = av;
}
```

### 私有结构
---
第二个解决方案是，将固定大小的对象放到私有结构体里，这样，Eigen对象会伴随类对象一起动态生成：
```cpp
struct Foo_d
{
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW
  Vector2d v;
  ...
};
struct Foo {
  Foo() { init_d(); }
  ~Foo() { delete d; }
  void bar()
  {
    // use d->v instead of v
    ...
  }
private:
  void init_d() { d = new Foo_d; }
  Foo_d* d;
};
```

这么做可见的好处是，Foo类不再需要为了解决对齐问题而刻意修改。但不足在于，你需要在堆分配的时候进行其他处理。

[1]:https://github.com  
[othersolution]:https://github.com  
[vector2d]:https://github.com  
[Eigen]:https://github.com  
[Eigen::VectorXd]:https://github.com  