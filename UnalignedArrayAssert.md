# 数组未对齐断言的解释

> ref: 
---
嘿！你会来看这一节的内容大概是因为你的程序出现了断言错误而终止运行的问题，像下面这个例子：
```
my_program: path/to/eigen/Eigen/src/Core/DenseStorage.h:44:
Eigen::internal::matrix_array<T, Size, MatrixOptions, Align>::internal::matrix_array()
[with T = double, int Size = 2, int MatrixOptions = 2, bool Align = true]:
Assertion `(reinterpret_cast<size_t>(array) & (sizemask)) == 0 && "this assertion
is explained here: http://eigen.tuxfamily.org/dox-devel/group__TopicUnalignedArrayAssert.html
     READ THIS WEB PAGE !!! ****"' failed.
```
已知会引起这个错误的原因有4种。请继阅读下去，了解这些原因然后学会如何解决这个错误。

## 在我的代码里究竟是哪里引发了这个错误？

首先，你需要做的是找出是哪一段代码引起了这个断言错误。大致扫一遍，这个错误信息似乎没什么帮助，它只是引导你去查看Eigen里的一份文档。即使你的程序发生了崩溃，如果你有办法复现这个问题，那么你可以借助随便哪个调试器来追溯到问题代码。例如，如果你使用的是gcc编译器，那么你可以像下面的例子这样使用gdb调试器来定位问题：
```
$ gdb ./my_program          # Start GDB on your program
> run                       # Start running your program
...                         # Now reproduce the crash!
> bt                        # Obtain the backtrace
```
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
那么，你需要阅读另一节：[包含Eigen对象作为成员变量的结构体][1]。
注意到，这里的[Eigen::Vector2d]只是一个例子,一般来说，这种问题通常会发生在所有[固定大小可量化的Eigen类型][2]里。

## 原因2：使用了STL容器或者手动申请对象内存

如果你使用了诸如std::vector，std::map等等stl容器来存放Eigen对象，或者使用了像下面这样成员变量包含有Eigen对象的类的时候
```c++
std::vector<Eigen::Matrix2f> my_vector;
struct my_class { ... Eigen::Matrix2f m; ... };
std::map<int, my_class> my_map;
```
你需要阅读另外一节：[在STL容器中使用Eigen](./UsingSTLContainersWithEigen.md)
请注意，这里的[Eigen::Matrix2f]()只是作为一个例子，更一般的，这种问题通常会发生在所有[固定大小可量化的Eigen类型][2]和[包含有Eigen对象作为成员变量的数据结构][1]里.

不使显示用`new`操作符来给上述类/方法分配内存与用户手动在类或者方法前使用`new`分配内存的场景都会引发上述问题。一个典型的例子是`std::make_shared`或者`std::allocate_shared`。问题的解决的方法是使用在[使用STL容器的方法](./UsingSTLContainersWithEigen.md)里介绍的[对齐的分配器]。

## 原因3：按值传递Eigen对象

如果你的代码里有下面这种按值传递[Eigen]对象的话
```c++ void func(Eigen::Vector4d v);```
你需要做的是阅读这一节：[在函数中按值传递Eigen对象](./PassingByValue.md)。
请注意到这里的[Eigen::Vector4d]只是一个例子，针对所有[固定大小可向量化的Eigen类型][2]都会出现这个问题。

## 原因4：编译器在栈对齐的时候推断错误（例如在windows环境下使用gcc编译器）

对于在windows环境下使用gcc编译器(像MinGW或者TDM-GCC)的人来说，这一节是必读章节。如果你的断言错误是由于使用了类似下列在方法中错误声明了局部变量的话：
```c++
void foo()
{
  Eigen::Quaternionf q;
  //...
}
```
你需要阅读这一节的内容：[编译器在栈对齐时推断错误](./WrongStackAlignment.md)。
注意到这里的[Eigen::Quaternionf]只是一个例子，这个问题会发生在所有[固定大小可量化的Eigen类型][2]上。

## 这个断言错误的一般解释

[固定大小可量化的Eigen对象][2]要求必须在16位对齐的地方创建,不然如果使用SIMD(单指令多字节数据)的编址方式来创建对象将导致崩溃。
一般情况下，[Eigen]会通过属性对齐或者重载`new`运算符来帮你处理这些对齐细节。
但是存在一些特殊场景，上述机制被重写了：这很可能是产生上述断言错误的原因。

## 我不在乎量优化的问题，我只想知道要如何解决这个错误？

三个可能的方案：
  - 在使用[Matrix],[Array],[Quaterntion]等等出现问题的时候使用DontAlign选项。这样，[Eigen]不会尝试对对象进行对齐，也不会预设任何特殊的对齐方式。带来的缺点是，你可能会为非对齐对象读写操作付出额外代价，但是在现代CPU上，这个代价通常不存在或者微小到几乎可以无视。[这里][1]是一个例子。
  - 定义[EIGEN_DONT_ALIGN_STATICALLY]()宏。这会禁用所有16位（或更多位）静态对齐的代码，但是堆的16位（或更高）的对齐会得到保留。这会导致确定大小的对象（例如Matrix4d)以非对齐方式量化存储（这由宏EIGEN_UNALIGNED_VECTORIZE控制），同时对于动态大小对象（像MatrixXd）的量化不改变。但是请注意这会破坏静态对齐带来的ABI特性。
  - 或者同时定义[EIGEN_DONT_VECTORIZE]()与EIGEN_DISABLE_UNALIGNED_ARRAY_ASSERT两个宏。这会保留16位对齐同时保留ABI特性，但是完全禁用量化。

如果你想知道为什么定义EIGEN_DONT_VECTORIZE不直接禁用16位对齐和这个断言，以下是解释：
我们没有禁用这个断言是因为原来禁用量化的机器可以运行的程序，在机器开启量化的时候程序会突然崩溃。我们没有禁用16位对齐，因为如果禁用会导致量化过和未量化的代码都无法兼容ABI特性。而ABI特性非常重要，哪怕是针对那些只开发自用应用的程序员来说，例如也许你会希望在同一个应用里分别使用量化路径和非量化路径。

[1]:./StructHavingEigenMembers.md
[2]:./FixedSizeVectorizable.md
[Eigen]:http://eigen.tuxfamily.org/dox/namespaceEigen.html