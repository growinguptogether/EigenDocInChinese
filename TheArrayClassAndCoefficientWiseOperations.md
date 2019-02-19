# 数组类和coefficient-wise操作

这一页提供如何使用Eigen数组类的简单说明。

## 什么是数组类？

Array类提供了通用的数组结构，而不是用于线性代数的Matrix类。
此外，Array类还提供了coefficient-wise的操作，该操作在线性系统中时没有意义的，
比如，每个数组中的元素分别依次相加，或分别依次相乘。

## 数组类型

Array是和Matrix具有相同模板参数的模板类。和Matrix一样，前三个模板参数是强制性需要的：

```c++
Array<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime>
```

后面三个参数是可选的，由于参数和Matrix是一致的，我们将不再进行额外的介绍，
具体可以参见：[矩阵类](./TheMatrixClass.md)

Eigen针对通用的情况给出了typedefs，相关的实现和Matrix的typedefs类似，但是有些许的不同，
一维而二维数组都是用“array”。我们采用ArrayNt的typedef形式来表示一维数组，
其中N表示数组的大小，t表示元素的类型，这和Matrix的typedefs是一样的。对于二维数组，
我们使用ArrayNNt的形式。下表中给出了一些示例：

| Type | Typedef |
|------|---------|
| `Array<float,Dynamic,1>` | ArrayXf |
| `Array<float,3>` | Array3f |
| `Array<double,Dynamic,Dynamic>` | ArrayXXd |
| `Array<double,3,3>` | Array33d |

## 访问Array中的元素

重载的括号操作符提供了array中元素的读写操作，这和matrix一样。
而且`<<`操作符同样可以用来初始化数组或打印数组。

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  ArrayXXf m(2,2);

  // 对数组中的元素进行赋值
  m(0,0) = 1.0; m(0,1) = 2.0;
  m(1,0) = 3.0; m(1,1) = m(0,1) + m(1,0);

  // 输出m
  cout << m << endl << endl;

  // 逗号初始化
  m << 1.0,2.0,3.0,4.0;

  // 输出m
  cout << m << endl;
}

// Output
1 2 
3 5 

1 2 
3 4
```

关于逗号初始化更多的介绍，见：TODO 增加链接

## 加法和减法

两个数组之间的相加和相减和矩阵是一致的。当数组大小一致的时候，这两个操作才是合理的，
同时这两个操作是coefficient-wise的。

数组也支持和标量进行上述运算，在执行`array + scalar`的时候是将标量加到数组的每一个元素上，
这个在Matrix对象上是不能直接进行的。

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  ArrayXXf a(3,3);
  ArrayXXf b(3,3);
  a << 1,2,3,
       4,5,6,
       7,8,9;
  b << 1,2,3,
       1,2,3,
       1,2,3;

  cout << "a + b = " << endl << a + b << endl << endl;
  
  cout << "a - 2 = " << endl << a - 2 << endl;
}
```

## 数组乘法


