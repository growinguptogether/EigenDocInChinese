# 编译器栈对齐推断错误
---
**这似乎是一个GCC的bug，这个问题已经在GCC4.5版本里修复了。如果你遇到了这个问题，请升级到GCC4.5版本，然后告诉我们问题的结果，好方便我们更新这一节的内容。**
目前为止，这个问题我们只在用GCC编译器（例如：MinGW和TDM-GCC）编译，运行在Windows平台上出现过。
像下面这个函数
```cpp
void foo()
{
  Eigen::Quaternionf q;
  //...
}
```
GCC默认认为函数栈已经是16字节对齐的了，对象q理所应当是在16字节对齐的位置构建。因此，编译器不会像[Eigen]所要求的那样，采取额外措施来确保q一定是在16字节对齐的位置上构建出来。
问题是，有一些特殊场景，这个假设在Windows平台下可能会不成立，此时，Windows仅保证栈满足4字节对齐。但是，即使GCC采取了措施保证了主函数里调用该函数时满足栈对齐，这仍不能保证其他线程或者其他编译器编译的机器码调用这个函数时，栈对齐还是可能会被破坏。结果就是对象"q"在一个不对齐的地方被构建出来，然后你的程序就因为[未对齐的数组断言](./UnalignedArrayAssert.md "assertion on unaligned arrays")错误而崩溃。到目前为止，我们找到了以下3种解决方案。

## 局部方案

局部方案是指在出错的函数前加上这个属性：
```cpp
__attribute__((force_align_arg_pointer)) void foo()
{
  Eigen::Quaternionf q;
  //...
}
```
想了解这段代码做了什么，你可以阅读[这篇](http://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Function-Attributes.html#Function-Attributes)GCC文档。这个操作只需要在使用GCC编译器同时生成的可执行文件在Windows平台上运行的场景下添加，因此，用一个宏来封装这些代码，让它在其他平台下无效会是个理想方案。这个方案的好处在于，你可以精心挑选出那些可能会破坏栈对齐的函数，针对性的处理。但是，麻烦在于你必须一个个处理这个类型的所有函数。所以你也可以考虑我们下面提的全局方案

## 全局方案

全局方案是，当你用GCC编译器编译Windows平台的可执行文件时，在你的工程属性中添加下面的GCC编译参数：
` -mincoming-stack-boundary=2 `
说明：这个参数告诉GCC编译器栈对齐仅仅保证2^2=4字节对齐，此时GCC知道它需要在[固定大小可量化Eigen类型](./FixedSizeVectorizable.md)需要的时候添加额外操作保证对象满足16字节对齐。

另一个全局方案是传递这个参数：
` -mstackrealign `
这跟在所有函数前添加`force_align_arg_pointer`属性操作效果是等价的。
这些全局方案使用起来很方便，但是请注意因为这些操作为每个函数都引入了额外的预处理/后处理指令，这可能导致程序运行速度的降低。

[Eigen]: http://eigen.tuxfamily.org/dox/namespaceEigen.html
