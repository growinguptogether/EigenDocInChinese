# 线性代数与分解

这一页介绍如何利用不同的分解方式对线性系统进行求解，如LU分解，QR分解，SVD分解。
在读完这一页之后，不要忘记稠密矩阵的分类。

## 基本线性求解

问题：你有一个方程系统，书写成矩阵方程如下：Ax = b.

其中A和b是矩阵，b可以是向量。求x。

求解方案：针对具体问题中的矩阵A，结合求解速度和精度，你可以在不同的分解方式中
选择出比较合适的一种。但是，先让我们来看下简单的例子：

```c++
// Example
#include <iostream>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

int main()
{
  Matrix3f A;
  Vector3f b;
  A << 1,2,3,  4,5,6,  7,8,10;
  b << 3, 3, 4;
  cout << "Here is the matrix A:\n" << A << endl;
  cout << "Here is the vector b:\n" << b << endl;
  Vector3f x = A.colPivHouseholderQr().solve(b);
  cout << "The solution is:\n" << x << endl;
}

// Output
Here is the matrix A:
 1  2  3
 4  5  6
 7  8 10
Here is the vector b:
3
3
4
The solution is:
-2
 1
 1
```

在这个例子中，方法`colPivHouseholderQr()`返回`ColPivHouseholderQR`类的对象。
由于矩阵的类型是Matrix3f，这一行可以被下面的式子替换：

```c++
ColPivHouseholderQR<>
```
