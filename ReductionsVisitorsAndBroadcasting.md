# Reductions, visitors 和 broadcasting

这一页介绍了Eigen的reductions，visitors，broadcasting
（这三个词翻译后会失去味道，因此保留），以及如何在矩阵和向量中使用。

## Reductions 

在Eigen里面，一个reductions，就是传入矩阵或向量，然后返回一个标量的函数。
最常用的一个reductions类型的函数是`.sum()`，返回矩阵或是向量的所有元素的和。

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;

int main()
{
  Eigen::Matrix2d mat;
  mat << 1, 2,
         3, 4;
  cout << "Here is mat.sum():       " << mat.sum() << endl;
  cout << "Here is mat.prod():      " << mat.prod() << endl;
  cout << "Here is mat.mean():      " << mat.mean() << endl;
  cout << "Here is mat.minCoeff():  " << mat.minCoeff() << endl;
  cout << "Here is mat.maxCoeff():  " << mat.maxCoeff() << endl;
  cout << "Here is mat.trace():     " << mat.trace() << endl;
}

// output
Here is mat.sum():       10
Here is mat.prod():      24
Here is mat.mean():      2.5
Here is mat.minCoeff():  1
Here is mat.maxCoeff():  4
Here is mat.trace():     5
```

`trace()`返回矩阵的迹，即矩阵的对角元素的和，这个和`a.diagonal().sum()`是等价的。

### 范数计算

> 译者加：关于范数可以参见：[向量范数和矩阵范数](https://blog.csdn.net/Michael__Corleone/article/details/75213123)

一个向量的2-范数（欧式距离）的平方可以通过`squaredNorm()`计算得到。
它等价于向量自己和自己的点积，也就是各个元素平方和。

Eigen也提供了`norm()`方法，获取`squaredNorm()`返回值的平方根。

这些操作也可以对矩阵进行操作，比如，一个n-p矩阵可以被当作大小为`n*p`的向量，
调用`norm()`得到Frobenius或Hilbert-Schmidt范数。这里我们避免使用2-范数，
因为这两个是不同的概念。

如果你想到得到基于coefficient-wise的范数，可以使用`lpNorm<p>()`方法。当是无穷范数的时候，
p可以是`Infinity`，即获取元素中绝对值最大的值。

下面给出了示例：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
	VectorXf v(2);
  MatrixXf m(2,2), n(2,2);

	v << -1, 2;
  
  m << 1, -2, 
       -3, 4;

  cout << "v.squaredNorm() = " << v.squaredNorm() << endl;
  cout << "v.norm() = " << v.norm() << endl;
  cout << "v.lpNorm<1>() = " << v.lpNorm<1>() << endl;
  cout << "v.lpNorm<Infinity>() = " << v.lpNorm<Infinity>() << endl;

  cout << endl;
  cout << "m.squaredNorm() = " << m.squaredNorm() << endl;
  cout << "m.norm() = " << m.norm() << endl;
  cout << "m.lpNorm<1>() = " << m.lpNorm<1>() << endl;
  cout << "m.lpNorm<Infinity>() = " << m.lpNorm<Infinity>() << endl;
}

// output
v.squaredNorm() = 5
v.norm() = 2.23607
v.lpNorm<1>() = 3
v.lpNorm<Infinity>() = 2

m.squaredNorm() = 30
m.norm() = 5.47723
m.lpNorm<1>() = 10
m.lpNorm<Infinity>() = 4
```

关于[矩阵1范数，无穷范数](https://en.wikipedia.org/wiki/Operator_norm)
可以通过下面的方式计算得到：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
  MatrixXf m(2,2);
  m << 1,-2,
       -3,4;

  cout << "1-norm(m)    = " << m.cwiseAbs().colwise().sum().maxCoeff() <<
       " == " << m.colwise().lpNorm<1>().maxCoeff() << endl;
  cout << "infy-norm(m) = " << m.cwiseAbs().rowwise().sum().maxCoeff() <<
       " == " << m.rowwise().lpNorm<1>().maxCoeff() << endl;
}

// output
1-norm(m)     = 6 == 6
infty-norm(m) = 7 == 7
```

下面会有更多表达式格式相关的介绍。

### 布尔reductions

下面的reduction操作是针对布尔的。

- `all()`如果矩阵或数组中的所有元素都可以评估为true，那么返回true
- `any()`如果矩阵或数组中有一个元素可以评估为true，那么返回true
- `count()`返回矩阵或数组中评估为true的元素的个数

这些操作通常与数组提供的coefficient-wise的比较操作和相等比较操作同时使用。如，
`array > 0`那么数组中所有大于零的元素的位置会被设置成true。因此，
`(array > 0).all()`用来检测数组中是不是所有元素都大于零。见如下例子：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
  ArrayXXf a(2,2);

  a << 1,2,
       3,4;

  cout << "(a > 0).all()    = " << (a > 0).all()   << endl;
  cout << "(a > 0).any()    = " << (a > 0).any()   << endl;
  cout << "(a > 0).count()  = " << (a > 0).count() << endl;
  cout << endl;
  cout << "(a > 2).all()    = " << (a > 2).all()   << endl;
  cout << "(a > 2).any()    = " << (a > 2).any()   << endl;
  cout << "(a > 2).count()  = " << (a > 2).count() << endl;
}

// output 
(a > 0).all()   = 1
(a > 0).any()   = 1
(a > 0).count() = 4

(a > 2).all()   = 0
(a > 2).any()   = 1
(a > 2).count() = 2<Paste>
```

### 用户自定义的reductions

TODO

在此期间可以看下`DenseBase::redux`函数。

## Visitors

visitors可用获取元素在数组或矩阵中的位置。

