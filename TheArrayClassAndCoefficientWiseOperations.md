# 数组类和coefficient-wise操作

> ref: http://eigen.tuxfamily.org/dox/group__TutorialArrayClass.html

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

// output
a + b = 
 2  4  6
 5  7  9
 8 10 12

a - 2 = 
-1  0  1
 2  3  4
 5  6  7
```

## 数组乘法

Array除了上述提供的操作之外，还提供了其他coefficient-wise的操作。比如，`.abs()`对每个元素求绝对值，
`.sqrt()`对每个元素求开方。如果你有两个相同大小的数组，可以调用`.min(.)`获取两个数组中对应元素的
最小值组成的数组。这些操作对应的例子如下：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  ArrayXf a = ArrayXf::Random(5);
  a *= 2;
  cout << "a = " << endl 
       << a << endl;
  cout << "a.abs() =" << endl
       << a.abs() << endl;
  cout << "a.abs().sqrt() =" << endl
       << a.abs().sqrt() << endl;
  cout << "a.min(a.abs().sqrt()) = " << endl
       << a.min(a.abs().sqrt()) << endl;
}

// output
a =
  1.36
-0.422
  1.13
  1.19
  1.65
a.abs() =
 1.36
0.422
 1.13
 1.19
 1.65
a.abs().sqrt() =
1.17
0.65
1.06
1.09
1.28
a.min(a.abs().sqrt()) =
  1.17
-0.422
  1.06
  1.09
  1.28
```

关于coefficient-wise操作的更多介绍可以见：TODO add link

## 数组和矩阵之间的表达式转换

那么什么时候使用Matrix类，什么时候Array类呢？针对Matrix的操作不能再Arrays上面执行，针对Array的
操作不能在Matrix上执行。因此，如果你需要进行线性代数相关的操作，那么使用Matrix操作，如果你需要做
coefficient-wise的操作，那么你需要使用数组。但是，当你需要对Matrix和Array同时进行操作的时候，
就不会这么简单了。这种情况下，你需要将Matrix转换成Array，或将Array转换成Matrix。这样，
不用管对象是Matrix，还是Array，都可以进行想要的操作。

Matrix表达式中有`.array()`方法，能够将Matrix转换成array表达式，
这样就能够执行coefficient-wise操作。反过来，Array表达式能够具有`.matrix()`方法。
正如所有的Eigen表达式抽象，编译器优化下，这样的操作不会有多少运行时消耗。`.array()`和`.matrix()`都可以被当做右值和左值。

Matrix和Array表达式混合使用在Eigen是禁止的。举个例子，你不能够直接将matrix加上array。但是，
它可以非常方便的通过`.array()`或`.matrix()`进行转换。这里面有个操作是例外的，就是赋值操作，
matrix表达式可以直接赋值给array，array表达式可以直接赋值给matrix。

接下来的例子说明了如何使用Matrix对象的`.array()`方法。如，`result = m.array() * n.array()`，
将m，n矩阵分别转换成array，然后进行coefficient-wise乘法，再赋值给result矩阵，这个是合理的，
因为，Eigen支持matrix和array之间的赋值操作。

事实上，由于类似上面的操作非常的常见，Eigen提供了`const .cwiseProduct(.)`来实现矩阵
coefficient-wise乘法操作。这个也在下面的例子中给出：

```c++
#include <Eigen/Dense>
#include <iostream>

using namespace Eigen;
using namespace std;

int main()
{
  MatrixXf m(2,2);
  MatrixXf n(2,2);
  MatrixXf result(2,2);

  m << 1,2,
       3,4;
  n << 5,6,
       7,8;

  result = m * n;
  cout << "-- Matrix m*n: -- " << endl
       << result << endl << endl;
  result = m.array() * n.array();
  cout << "-- Array m*n: -- " << endl
       << result << endl << endl;
  result = m.cwiseProduct(n);
  cout << "-- With cwiseProduct: -- " << endl
       << result << endl << endl;
  result = m.array() + 4;
  cout << "-- Array m + 4: -- << endl
       << result << endl << endl;
}

// output
-- Matrix m*n: --
19 22
43 50

-- Array m*n: --
 5 12
21 32

-- With cwiseProduct: --
 5 12
21 32

-- Array m + 4: --
5 6
7 8
```

类似的，如果array1和array2是数组，`array1.matrix() * array2.matrix()`就是进行矩阵乘法操作。

这里有个更加高阶的例子。`(m.array() + 4).matrix() * m)`这个表达式，先对矩阵m的各个元素加4，
然后再与矩阵m进行矩阵乘法。类似的，`(m.array() * n.array()).matrix * m`这个表达式，
先将矩阵m和n执行coefficient-wise乘法，然后再与矩阵m执行矩阵乘法。

```c++
#include <Eigen/Dense>
#include <iostream>
using namespace Eigen;
using namespace std;
int main()
{
  MatrixXf m(2,2);
  MatrixXf n(2,2);
  MatrixXf result(2,2);
  m << 1,2,
       3,4;
  n << 5,6,
       7,8;
  
  result = (m.array() + 4).matrix() * m;
  cout << "-- Combination 1: --" << endl << result << endl << endl;
  result = (m.array() * n.array()).matrix() * m;
  cout << "-- Combination 2: --" << endl << result << endl << endl;
}

// output
-- Combination 1: --
23 34
31 46

-- Combination 2: --
 41  58
117 170
```


