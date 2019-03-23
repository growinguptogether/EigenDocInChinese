# 将Eigen对象用值传递参数传递方式传给函数
---

在C++中将对象用值传递方式给函数传参是一个非常坏的编码方式，相比于引用传递，这意味着额外的无意义的拷贝构造操作。
在使用[Eigen]的场景下，这种问题会更突出：用值传递方式传递[固定大小可量化的Eigen对象](./FixedSizeVectorizable.md)将不仅仅是效率更低，这甚至可能是非法的然后导致你的程序崩溃！原因是，处理这些[Eigen]对象的对齐操作的机制在值传递的时候不会生效。
所以，像下面这个函数这样按值传递v：
```cpp 
void my_function(Eigen::Vector2d v); 
```
必须改成下面这种引用传递v：
```cpp
void my_function(const Eigen::Vector2d& v);
```
类似的，如果你的类里面包含了[Eigen]对象作为成员变量：
```cpp
struct Foo
{
  Eigen::Vector2d v;
};
void my_function(Foo v);
```
这个函数也需要像下面这样重写：
```cpp
void my_function(const Foo& v);
```
请注意，Eigen对象作为函数返回值返回不会有任何问题。
[Eigen]:http://eigen.tuxfamily.org/dox/namespaceEigen.html
