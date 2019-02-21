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

visitors可用获取元素在数组或矩阵中的位置。最简单的使用方式是：`maxCoeff(&x,&y)`和`minCoeff(&x,&y)`，这两个函数是用来找到矩阵或数组中最大元素和最小元素的位置。

传递给visitor的参数，存储行列位置的变量的指针。这些变量的类型应该是`Index`，如下：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
  MatrixXf m(2,2);

  m << 1,2,
       3,4;

  // get location of maximum
  MatrixXf::Index maxRow, maxCol;
  float max = m.maxCoeff(&maxRow, &maxCol);

  // get location of minimum
  MatrixXf::Index minRow, minCol;
  float min = m.minCoeff(&minRow, &minCol);

  cout << "Max: " << max << ", at: " << maxRow << "," << maxCol << endl;
  cout << "Min: " << min << ", at: " << minRow << "," << minCol << endl;
}

// output
Max: 4, at: 1,1
Min: 1, at: 0,0
```

同时两个函数也分别返回了最大值和最小值。

## 部分reductions

部分reductions是针对矩阵或向量的行或列进行reductions操作，并且返回一个行或列向量。
部分reductions和`colwise()`或者`rowwise()`一起配合使用。

一个简单的获取矩阵每一列最大值元素，并将结果存储到一个行向量的例子如下：

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;

int main()
{
  Eigen::MatrixXf mat(2,4);
  mat << 1,2,6,9,
         3,1,7,2;
  cout << "Column's maximum: " << endl
       << mat.colwise().maxCoeff() << endl;
}

// output
Column's maximum: 
3 2 7 9
```

`rowwise()`的例子如下：

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;
int main()
{
  Eigen::MatrixXf mat(2,4);
  mat << 1,2,6,9,
         3,1,7,2;
  cout << "Row's maximum: " << endl
       << mat.rowwise().maxCoeff() << endl;
}

// output
Row's maximum: 
9
7
```

需要注意的是，`column-wise`操作返回的是行向量，而`row-wise`返回的是列向量。

### 将部分reductions和其他操作组合

这里也可以将部分reductions的结果进行进一步处理。下面是找到矩阵中列方向的元素和最大的列：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace std;
using namespace Eigen;

int main()
{
  MatrixXf mat(2,4);
  mat << 1,2,6,9,
         3,1,7,2;
  
  MatrixXf::Index maxIndex;
  float maxNorm = mat.colwise().sum().maxCoeff(&maxIndex);
  cout << "Maximum sum at position " << maxIndex << endl;

  cout << "The corresponding vector is: " << endl;
  cout << mat.col(maxIndex) << endl;
  cout << "And its sum is: " << maxNorm << endl;
}

// output
Maximum sum at position 2
The corresponding vector is: 
6
7
And its sum is: 13
```

前面的例子，使用colwise()，然后再使用sum(),进行降维，得到一个1x4大小的新矩阵。

因此，如果

![](http://latex.codecogs.com/gif.latex?m%20%3D%20%5Cbegin%7Bbmatrix%7D%201%20%26%202%20%26%206%20%26%209%5C%5C%203%20%26%201%20%26%207%20%26%202%20%5Cend%7Bbmatrix%7D)

那么：

![](http://latex.codecogs.com/gif.latex?m.colwise%28%29.sum%28%29%20%3D%20%5Cbegin%7Bbmatrix%7D%204%20%26%203%20%26%2013%20%26%2011%20%5Cend%7Bbmatrix%7D)

`maxCoeff()`就能够获取列和最大的位置。

## Broadcasting

broadcasting背后的概念和部分reductions比较类似，不同的地方是，
broadcasting将一个向量（行向量或列向量）通过一个方向上的扩展构造成了一个矩阵。

一个简单的例子是将某个列向量加到矩阵的每一列上。如下：

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;

int main()
{
  Eigen::MatrixXf mat(2,4);
  Eigen::VectorXf v(2);

  mat << 1,2,6,9,
         3,1,7,2;
  v << 0,
       1;
  
  // add v to each column of m
  mat.colwise() += v;

  std::cout << "Broadcasting result: " << std::endl;
  std::cout << mat << std::endl;
}

// output
Broadcasting result: 
1 2 6 9
4 2 8 3
```

我们可以将`mat.colwise() += v`这个指令，用两种方式进行理解。一种是，它将vector v加到矩阵的每一列。
另一种理解是，先将向量v在行方向上进行扩展，形成2x4的矩阵，然后在进行求和：

![](http://latex.codecogs.com/gif.latex?%5Cbegin%7Bbmatrix%7D%201%20%26%202%20%26%206%20%26%209%20%5C%5C%203%20%26%201%20%26%207%20%26%202%20%5Cend%7Bbmatrix%7D%20&plus;%20%5Cbegin%7Bbmatrix%7D%200%20%26%200%20%26%200%20%26%200%20%5C%5C%201%20%26%201%20%26%201%20%26%201%20%5Cend%7Bbmatrix%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%201%20%26%202%20%26%206%20%26%209%20%5C%5C%204%20%26%202%20%26%208%20%26%203%20%5Cend%7Bbmatrix%7D)

`-=`，`+`和`-`操作符一样可以使用到column-wise和row-wise。针对数组，我们还可以使用
`*=`，`/=`，`*`用来执行coefficient-wise的column-wise或row-wise的乘法和除法操作。
这些操作在矩阵上面是不适用的，因为矩阵上这样的操作对矩阵而言是不清晰的。
如果你想要将v(0)和mat矩阵的第0列相乘，v(1)和mat矩阵的第一列相乘，等等，那么可以使用`mat = mat * v.asDiagonal()`。

值得注意的是能够被按column-wise或row-wise进行相加的必定是向量类型，而不能是矩阵。
如果使用矩阵，那么会得到编译器错误。这也意味这，broadcasting操作仅仅能用在向量而不能用在矩阵。
对于Array类，也是一样的，只能够用在ArrayXf，而不能用在多维数组。

对于行的扩展如下：

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;

int main()
{
  Eigen::MatrixXf mat(2,4);
  Eigen::VectorXf v(4);

  mat << 1,2,6,9,
         3,1,7,2;

  v << 0,1,2,3;
  
  mat.rowwise() += v.transpose();
  cout << "Broadcasting result: " << endl;
  cout << mat << endl;
}

// output 
Broadcasting result: 
 1  3  8 12
 3  2  9  5
```

### 将broadcasting和其它操作一起使用

broadcasting操作也一样可以和其他操作配合使用，如Matrix或Array的操作，
reductions或是部分reductions。

到目前为止，broadcasting，reductions和部分reductions都已经进行了介绍，下面给出一个
更加高阶的例子，寻找矩阵的各个列中和向量v最接近的列。此处，会使用欧几里德距离，
使用部分reduction函数`squaredNorm()`进行计算欧式距离的平方。

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace std;
using namespace Eigen;

int main()
{
  MatrixXf m(2,4);
  VectorXf v(2);

  m << 1, 23, 6, 9,
       3, 11, 7, 2;
  
  v << 2,
       3;

  MatrixXf::Index index;
  // find nearest neighbour
  (m.colwise() - v).colwise().squaredNorm().minCoeff(&index);

  cout << "Nearest neighbour is column " << index << ":" << endl;
  cout << m.col(index) << endl;
}
```

主要完成任务的代码为：

```c++
(m.colwise() - v).colwise().squaredNorm().minCoeff(&index);
```

我们按照以下步骤对其进行理解：

- m.colwise() - v 是一个broadcasting操作，从m每一列中将v减掉。
得到的新的矩阵和原来的矩阵m有相同的大小：

![](http://latex.codecogs.com/gif.latex?m.colwise%28%29-v%3D%5Cbegin%7Bbmatrix%7D%20-1%20%26%2021%20%26%204%20%26%207%20%5C%5C%200%20%26%208%20%26%204%20%26%20-1%20%5Cend%7Bbmatrix%7D)

- (m.colwise() - v).colwise().squaredNorm() 是一个局部reduction，用来计算列方向上的和的平方。
得到的结果是行向量，每个元素表示m的列和v之间的欧几里德距离的平方。

![](http://latex.codecogs.com/gif.latex?%28m.colwise%28%29-v%29.colwise%28%29.squaredNorm%28%29%20%3D%20%5Cbegin%7Bbmatrix%7D%201%20%26%20505%20%26%2032%20%26%2050%20%5Cend%7Bbmatrix%7D)

- 最后，minCoeff(&index)用来获取距离最小的列。
