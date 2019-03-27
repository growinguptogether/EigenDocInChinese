# 参数排序的数学函数目录
---
下表是Eigen支持的数学函数目录。表中，a,b指代[Array]对象或表达式,m指代线性代数矩阵/向量对象。普通标量数据类型的缩写如下：
- int: i32  
- float: f  
- double: d  
- std::complex<float>: cf  
- std::complex<double>: cd  

在每一行里，第一列列出了数组（矩阵如果可用）方法的几种等价调用形式。如果你使用`m.array()`将矩阵转换为数组，那么它将可以使用所有数组拥有的方法。
第三列是方法的基础实现提示。大多数场景下，Eigen采用STL提供的实现处理标准变量类型，采用用户提供的实现处理自定义变量类型。例如，在简单调用STL方法的同时，保留根据[参数映射表](http://en.cppreference.com/w/cpp/language/adl)来调用自定义实现的功能。像下面这样：
```cpp
using std::foo;
foo(a[i]);
```
意味着如果是基础变量类型就调用STL的实现。如果不是，那么用户就得确保对于指定的变量类型他们提供了可访问的重载函数foo（通常定义在变量相同的命名空间下）。这同时意味着，如果方法`std::foo`仅仅在某些较新的c\++版本（例如c\++11）里实现了，那么只有在编译器支持必需的编译器版本的情况下，对应的方法才是可用的。

[Array]: http://eigen.tuxfamily.org/dox/classEigen_1_1Array.html