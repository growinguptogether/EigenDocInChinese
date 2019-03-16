# 数组未对齐断言的解释

> ref: 
---
嘿！你会来看这一节的内容大概是因为你的程序出现了断言错误而终止运行的问题，像下面这个例子：
my_program: path/to/eigen/Eigen/src/Core/DenseStorage.h:44:
Eigen::internal::matrix_array<T, Size, MatrixOptions, Align>::internal::matrix_array()
[with T = double, int Size = 2, int MatrixOptions = 2, bool Align = true]:
Assertion `(reinterpret_cast<size_t>(array) & (sizemask)) == 0 && "this assertion
is explained here: http://eigen.tuxfamily.org/dox-devel/group__TopicUnalignedArrayAssert.html
     READ THIS WEB PAGE !!! ****"' failed.

已知会引起这个错误的原因有4种。请继阅读下去，了解这些原因然后学会如何解决这个错误。

## 在我的代码里究竟是哪里引发了这个错误？

首先，你需要做的是找出是哪一段代码引起了这个断言错误。大致扫一遍，这个错误信息似乎没什么帮助，它只是引导你去查看Eigen里的一份文档。即使你的程序发生了崩溃，如果你有办法复现这个问题，那么你可以借助随便哪个调试器来追溯到问题代码。例如，如果你使用的是gcc编译器，那么你可以像下面的例子这样使用gdb调试器来定位问题：
`
$ gdb ./my_program          # Start GDB on your program
> run                       # Start running your program
...                         # Now reproduce the crash!
> bt                        # Obtain the backtrace
`
现在，你准确定位到引发这个问题的代码了，继续阅读下面的内容，了解你需要如何修复这个错误。

## 原因1：结构体的成员变量包含了Eigen对象

如果你的代码与下面相似：
```c++
class Foo
{
  //...
  Eigen::Vector2d v;
  //...
};
//...
Foo *foo = new Foo;
```
那么，你需要阅读另一节：[包含Eigen对象作为成员变量的结构体]()。
注意到，这里的[Eigen::Vector2d]只是一个例子,一般来说，这种问题通常会发生在所有[固定大小可向量化的Eigen类型]里。

## 原因2：使用了STL容器或者手动申请对象内存

如果你使用了诸如std::vector，std::map等等stl容器来存放Eigen对象，或者使用了像下面这样成员变量包含有Eigen对象的类的时候
```c++
std::vector<Eigen::Matrix2f> my_vector;
struct my_class { ... Eigen::Matrix2f m; ... };
std::map<int, my_class> my_map;
```
你需要阅读另外一节：[在STL容器中使用Eigen]()
请注意，这里的[Eigen::Matrix2f]()只是作为一个例子，更一般的，这种问题通常会发生在所有[固定大小可向量化的Eigen类型]()和[包含有Eigen对象作为成员变量的数据结构]()里.