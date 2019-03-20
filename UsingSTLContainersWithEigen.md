# 在STL容器中使用Eigen对象
---

## 总结

在STL容器中使用[固定大小可量化的Eigen对象类型 ](./FixedSizeVetorizableEigenObjects.md)，或者使用包含有Eigen对象的类的时候，你需要执行以下两个步骤：
- 必须使用16字节对其的内存分配器。Eigen提供了`aligned_allocator`供你使用。
- 如果你想使用`std::vector`容器，你必须加上`#include <Eigen/stdVector>`。

只有在存在[固定大小可量化Eigen类型](./FixedSizeVectorizableEigenObjects.md)或者[包含有Eigen对象作为成员变量的结构体](./StructHavingEigenMembers.md)的时候才存在以上问题。对于其他的Eigen类型，例如Vector3f或者MatrixXd，在使用STL容器的时候并不需要特别的操作。

## 使用对其的内存分配器

STL容器的模板可以接受内存分配器类型作为一个可选参数。当在STL容器中使用[固定大小可量化的Eigen对象类型 ](./FixedSizeVetorizableEigenObjects.md)时，你需要告诉容器使用始终采取在16字节整数倍的地址开辟内存空间的内存分配器。好消息是，Eigen已经为你准备好了这种分配器：`Eigen::aligned_allocator`。

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


