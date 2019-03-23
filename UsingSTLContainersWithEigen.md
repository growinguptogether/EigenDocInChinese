# 在STL容器中使用Eigen对象
---

## 总结

在STL容器中使用[固定大小可量化Eigen对象类型][1]，或者使用包含有Eigen对象的类的时候，你需要执行以下两个步骤：
- 必须使用16字节对其的内存分配器。Eigen提供了`aligned_allocator`供你使用。
- 如果你想使用`std::vector`容器，你必须加上`#include <Eigen/stdVector>`。

只有在存在[固定大小可量化Eigen对象类型][1]或者[包含有Eigen对象作为成员变量的结构体](./StructHavingEigenMembers.md)的时候才存在以上问题。对于其他的Eigen类型，例如Vector3f或者MatrixXd，在使用STL容器的时候并不需要特别的操作。

## 使用对其的内存分配器

STL容器的模板可以接受内存分配器类型作为一个可选参数。当在STL容器中使用[固定大小可量化Eigen对象类型][1]时，你需要告诉容器使用始终采取在16字节整数倍的地址开辟内存空间的内存分配器。好消息是，Eigen已经为你准备好了这种分配器：`Eigen::aligned_allocator`。

例如，你的代码不应该是：
```cpp
std::map<int,Eigen::Vector4f>
```
你需要这样写：
```cpp
std::map<int,Eigen::Vector4f,std::less<int>,
        Eigen::aligned_allocator<std::pair<const int, Eigen::Vector4f> > >
```

注意第三个参数`std::less<int>`这个参数是使用的默认值，但是因为我们需要指定第四个参数（分配器类型），我们不得不显示指定这个默认值。

## std::vector的场景

当我们使用std::vector的时候这个问题会更严重（原因稍后解释），所以我们必须明确指定使用[Eigen::aligned_allocator][allocator]处理内存分配。在实际应用中，你**必须**使用使用[Eigen::aligned_allocator][allocator]（不可以使用其他分配器），**同时**添加引用`#include <Eigne/StdVector>`.
范例如下：
```cpp
#include <Eigen/StdVector>
/* ... */
std::vector<Eigen::Vector4f,Eigen::aligned_allocator<Eigen::Vector4f>>
```

## 替代方案——让std::vector适配Eigen类型
---
推荐的替代方法已经在上文讲过了，std::vector提供了选项让你指定[Eigen]需要的对齐分配器。这么做的好处是，你不需要在所有使用std::vector的地方都指定Eigen::allocator。但是不便的一面是，你需要在每个类似于`std::vector<Vector2d>`的代码前加上指定分配器的声明。不然，编译器不知道指定的分配器，它将采用默认的std::allocator进行内存分配，然后你的程序很可能会崩溃。
范例如下：
```cpp
#include <Eigen/StdVector>
/* ... */
EIGEN_DEFINE_STL_VECTOR_SPECIALIZATION(Matrix2d)
std::vector<Eigen::Vector2d>
```
**解释**：std::vector的resize()方法使用了Value_type声明（默认使用Value_type())。所以，以std::vector<Eigen::Vector4f>为例，一些Eigen::Vector4f对象会按值传递，这将造成对齐机制被绕过，最终导致在一个非对齐的地址上创建Eigen::Vector4f对象。为了避免上述情况，我们发现唯一的解决方案只有特化std::vector，让它为Eigen::Vector4f做一些小改变，然后就能很好的解决这个问题了。

[1]:./FixedSizeVetorizableEigenObjects.md
[Eigen]:https://www.bing.com
[allocator]:https://www.bing.com
