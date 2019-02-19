# 块操作

> ref: http://eigen.tuxfamily.org/dox/group__TutorialBlockOperations.html

这一章来介绍块操作的要点。块指的是矩阵或数组中的矩形区域。块表达式可以当作左值，
也可以当作是右值。和Eigne其他的表达式一样，这些抽象几乎不消耗运行时间。

## 使用块操作

Eigen中最常用的块操作是`.block()`。它有两个版本分别如下：

| 块操作 | 具有动态大小的块表达式 | 具有固定大小的块表达式 |
|--------|------------------------|------------------------|
| 从点(i,j)开始，大小为(p,q)的块 | `matrix.block(i,j,p,q);` | `matrix.block<p,q>(i,j);` |

Eigen中所有的indices从0开始。

两个版本都可以使用在matrix和array。这两个表达式在语义上是一致的。
唯一的区别是，当块的大小比较小的时候，固定大小的版本能够给你执行更快的代码，
但是这个大小在编译时，需要是已知的。

下面的程序使用了动态大小和固定大小的版本来打印矩阵块：

```c++
#include <Eigen/Dense>
#include <iostream>

int main()
{
  Eigen::MatrixXf m(4,4);
  m << 1, 2, 3, 4,
       5, 6, 7, 8,
       9, 10, 11, 12,
       13, 14, 15, 16;
  cout << "Block in the middle" << endl;
  cout << m.block<2,2>(1,1) << endl << endl;
  for (int i=1; i<=3; ++i)
  {
    cout << "Block of size " << i << "x" << i << endl;
    cout << m.block(0,0,i,i) << endl << endl;
  }
}

// output
Block in the middle
 6  7
10 11

Block of size 1x1
1

Block of size 2x2
1 2
5 6

Block of size 3x3
 1  2  3
 5  6  7
 9 10 11
```

上述例子中`.block()`函数返回值作为右值。但是，它也可以作为左值，
意味着可以将数据赋值到块。

这一点下面的例子中给出。同时也证实了，数组中的块和矩阵中的块使用上的一致性。

```c++
#include <Eigen/Dense>
#include <iostream>

int main()
{
  Array22f m;
  m << 1,2,
       3,4;
  Array44f a = Array44f::Constant(0.6);
  cout << "Here is the array a:" << endl << a << endl << endl; 
  a.block<2,2>(1,1) = m;
  cout << "Here is now a with m copied into its central 2x2 block:"
    << endl << a << endl << endl;
  a.block(0,0,2,3) = ablock(2,1,2,3);
  cout << "Here is now a with bottom-right 2x3 block copied into top-left 2x2 block:"
    << endl << a << endl << endl;
}

// output 
Here is the array a:
0.6 0.6 0.6 0.6
0.6 0.6 0.6 0.6
0.6 0.6 0.6 0.6
0.6 0.6 0.6 0.6

Here is now a with m copied into its central 2x2 block:
0.6 0.6 0.6 0.6
0.6   1   2 0.6
0.6   3   4 0.6
0.6 0.6 0.6 0.6

Here is now a with bottom-right 2x3 block copied into top-left 2x2 block:
  3   4 0.6 0.6
0.6 0.6 0.6 0.6
0.6   3   4 0.6
0.6 0.6 0.6 0.6
```

尽管`.block()`方法提供了任意的block操作，但是还有一些特殊的实现方式，
提供了更加特殊的API或提供了更好的性能。关于性能，在编译期间提供给Eigen更多的信息，
就能获得更好的运行性能。比如，如果一个块是矩阵中的一列，那么可以使用`.col()`函数，
这给了Eigen提供优化的机会。

接下去的内容对这些特殊的方法进行了介绍。

## 列和行

独立的列和行是特殊的块。Eigen提供了如下的方法：

| 块操作 | 方法 |
|--------|------|
| 第i行 | `matrix.row(i);` |
| 第j列 | `matrix.col(j);` |

`col()`和`row()`的参数是，能够被访问到的行和列。Eigen中所有indices都是0开始的。

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  MatrixXf m(3,3);
  m << 1,2,3,
       4,5,6,
       7,8,9;
  cout << "Here is the matrix m:" << endl;
  cout << "2nd Row:" << m.row(1) << endl;
  m.col(2) += 3*m_col(0);
  cout << "After adding 3 times the first column into the third column, the matrix m is:\n";
  cout << m << endl;
}

// output 
Here is the matrix m:
1 2 3
4 5 6
7 8 9
2nd Row: 4 5 6
After adding 3 times the first column into the third column, the matrix m is:
 1  2  6
 4  5 18
 7  8 30
```

这个例子也说明了块操作可以像其他任意操作一样使用。

## 与边角相关的操作

Eigen提供了特殊的接口用来获取矩阵或数组的边角上的块。
如，`.topLeftCorner()`用来获取矩阵左上角的块。

不同的形式在下表中给出了小结：

| 块操作 | 动态大小的操作 | 固定大小的操作 |
|--------|----------------|----------------|
| pq大小左上角块 | `matrix.topLeftCorner(p,q)` | `matrix.topLeftCorner<p,q>()` |
| pq大小左下角块 | `matrix.bottomLeftCorner(p,q)` | `matrix.bottomLeftCorner<p,q>()` |
| pq大小右上角块 | `matrix.topRightCorner(p,q)` | `matrix.topRightCorner<p,q>()` |
| pq大小右下角块 | `matrix.bottomRightCorner(p,q)` | `matrix.bottomRightCorner<p,q>()` |
| 上面q行 | `matrix.topRows(q)` | `matrix.topRows<q>()` |
| 下面q行 | `matrix.bottomRows(q)` | `matrix.bottomRows<q>()` |
| 左边p列 | `matrix.leftCols(p)` | `matrix.leftCols<p>()` |
| 右边p列 | `matrix.rightCols(p)` | `matrix.rightCols<p>()` |

下面给出了使用上面操作的例子：

```c++
#include<Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  Matrix4f m;
  m << 1,2,3,4,
       5,6,7,8,
       9,10,11,12,
       13,14,15,16;
  cout << "m.leftCols(2) =" << endl << m.leftCols(2) << endl << endl;
  cout << "m.bottomRows<2>() =" << endl << m.bottomRows<2>() << endl << endl;
  m.topLeftCorner(1,3) = m.bottomRightCorner(3,1).transpose();
  cout << "After assignment, m = " << endl << m << endl;
}

// output
m.leftCols(2) =
 1  2
 5  6
 9 10
13 14

m.bottomRows<2>() =
 9 10 11 12
13 14 15 16

After assignment, m = 
 8 12 16  4
 5  6  7  8
 9 10 11 12
13 14 15 16
```

## 向量的区块操作

Eigen针对向量和一维的数组，也提供了一些块操作。

| 块操作 | 动态大小的操作 | 固定大小的操作 |
|--------|----------------|----------------|
| 前n个元素 | `vector.head(n)` | `vector.head<n>()` |
| 后n个元素 | `vector.tail(n)` | `vector.tail<n>()` |
| 从i开始的n个元素 | `vector.segment(i,n)` | `vector.segment<n>(i)` |

示例如下：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  ArrayXf v(6);
  v << 1,2,3,4,5,6;
  cout << "v.head(3) =" << endl << v.head(3) << endl << endl;
  cout << "v.tail<3>() =" << endl << v.tail<3>() << endl << endl;
  v.segment(1,4) *= 2;
  cout << "After `v.segment(1,4) *= 2`, v =" << endl << v << endl;
}

// output 
v.head(3) =
1
2
3

v.tail<3>() = 
4
5
6

after 'v.segment(1,4) *= 2', v =
 1
 4
 6
 8
10
 6
```
