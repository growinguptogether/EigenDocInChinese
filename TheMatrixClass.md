# 矩阵类

> ref: http://eigen.tuxfamily.org/dox/group__TutorialMatrixClass.html

在Eigen，所有的矩阵和向量都是Matrix模板类对象。向量1行或1列的特殊矩阵。

## Matrix的前三个模板参数

Matrix类有6个模板参数，但是就现在而言掌握前三个参数已经足够了。剩下的三个参数，带有默认值，在后面的内容中会进行讨论。

Matrix前三个强制的模板参数是：

`Matrix<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime>`

- Scalar, 是标量类型，如每个元素的类型。也就是说，如果你想要一个存储float的矩阵，那么scalar就是float。
- RowsAtCompileTime和ColsAtCompileTime是在编译期间需要确定的矩阵的行数和列数。

我们提供了很多方便的typedef来覆盖大多数场景。比如，`Matrix4f`是一个4x4的float矩阵，在eigen中是如下定义的：

`typedef Matrix<float,4,4> Matrix4f;`

接下来会讨论一些通用的typedefs。

## Vectors

正如上面所说，Vector就是矩阵的一个特例。在Eigen，以Vector开头的都是列向量。

举个例子，Vector3f就是3个float元素的列向量，在Eigen中定义如下：

`typedef Matrix<float,3,1> Vector3f;`

对于行向量也有类似的定义，如下：

`typedef Matrix<int, 1, 2> RowVector2i;`

## Dynamic特殊值

当然，Eigen并没有限制编译的时候必须知道矩阵的大小。RowsAtCompileTime和ColsAtCompileTime模板参数可以设定为特殊的值，`Dynamic`,用来表示编译期间大小是未知的。在Eigen中，这样的大小被叫做动态大小，在编译期间确定的大小被叫做固定大小。以MatrixXd为例，如下：

`typedef Matrix<double, Dynamic, Dynamic> MatrixXd;`

类似的定义VectorXi，如下：

`typedef Matrix<int, Dynamic, 1> VectorXi;`

当然也可以定义固定行数，不固定列数，如：`Matrix<float, 3, Dynamic>`.

## 构造函数

通常提供默认构造函数，默认构造函数不会进行内存分配，也不会进行矩阵元素的初始化。可以如下：

```c++
Matrix3f a;
MatrixXf b;
```

这里：

- a，是一个3x3的矩阵，一个没有初始化的float[9]数组
- b不固定大小的矩阵，当前大小为0x0，元素数组还没有被分配空间。

还有带大小的构造函数。对于矩阵而言，第一个参数通常是行数。对于向量而言，只需要传入向量的大小。构造函数内部会根据大小分配对应的内存空间，但是并不会对元素进行初始化。

```c++
MatrixXf a(10,15);
VectorXf b(30);
```

这里：

- a是一个10x15大小可以动态设定的矩阵，内存空间已经分配，但是还没有进行初始化。
- b是一个大小为30的可以动态设定大小的向量，内存空间已经分配，但是还没有进行初始化。

为了使得固定大小和动态设定大小的矩阵的接口保持一致，对于固定大小的矩阵，构造的时候传入大小也是合法的。如：

`Matrix3f a(3,3);`

最后，我们也提供了一些方便固定大小的vector初始化的构造，如下：

```
Vector2d a(5.0, 6.0);
Vector3d b(5.0, 6.0, 7.0);
Vector4d c(5.0, 6.0, 7.0, 8.0);
```

## 元素访问

Eigen中主要元素的访问和修改，是通过对括号进行了重载实现的。对于矩阵而言，第一个参数总是行的index。对于向量而言，只需要传递一个参数即可。index是从0开始计数的。例子如下：

```c++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;
int main()
{
  MatrixXd m(2,2);
  m(0,0) = 3;
  m(1,0) = 2.5;
  m(0,1) = -1;
  m(1,1) = m(1,0) + m(0,1);
  std::cout << "Here is the matrix m:\n" << m << std::endl;
  VectorXd v(2);
  v(0) = 4;
  v(1) = v(0) - 1;
  std::cout << "Here is the vector v:\n" << v << std::endl;
}
```

输出的结果为：

```
Here is the matrix m:
  3  -1
2.5 1.5
Here is the vector v:
4
3
```

注意单个参数的方式（`m(index)`）并不是一定针对向量，也可以用于矩阵，用于通过index来访问内部存储的一维数组中对应的元素。但是这个得到的结果依赖于矩阵的存储方式。Eigen默认采用的是以列为主的存储方式，不过也可以通过参数的设置改变成以行为主的存储方式。

`[]`操作符也用于向量元素的访问，不过需要注意的是，c++不允许操作符`[]`传入多于一个的参数。因此该操作符仅能用于向量的访问。

## 逗号初始化

矩阵和向量能够很方便的通过逗号初始化的方式进行初始化，如下：

```
Matrix3f m;
m << 1, 2, 3,
     4, 5, 6,
     7, 8, 9;
std::cout << m;

// output is:
1 2 3
4 5 6
7 8 9
```

## 改变大小

矩阵当前的大小可以通过`rows(),cols(),size()`获取。这些方法返回矩阵的行数，列数，以及元素的个数。对于大小动态设定的矩阵的resize，可以通过函数`resize()`实现。如下：

```c++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;
int main()
{
  MatrixXd m(2,5);
  m.resize(4,3);
  std::cout << "The matrix m is of size "
            << m.rows() << "x" << m.cols() << std::endl;
  std::cout << "It has " << m.size() << " coefficients" << std::endl;
  VectorXd v(2);
  v.resize(5);
  std::cout << "The vector v is of size " << v.size() << std::endl;
  std::cout << "As a matrix, v is of size "
            << v.rows() << "x" << v.cols() << std::endl;
}

// 结果：
The matrix m is of size 4x3
It has 12 coefficients
The vector v is of size 5
As a matrix, v is of size 5x1
```

如果矩阵实际的元素个数（size）没有发生变化的话，那么resize操作不会改变元素的数值，否则，该操作是破坏性的：元素的数值很有可能被更改。如果你想要在resize的时候不改变元素的数值，可以选用另一个方法，`conservativeResize()`。

当然这些接口对于固定大小的矩阵也能够调用，但是，你并不能改变矩阵的大小。试图改变，会引起程序的中断，但是如果大小没有发生变化，那么代码是合理的。如：

```c++
#include <iostream>
#include <Eigen/Dense>
using namespace Eigen;
int main()
{
  Matrix4d m;
  m.resize(4,4); // no operation
  std::cout << "The matrix m is of size "
            << m.rows() << "x" << m.cols() << std::endl;
}

// output
The matrix m is of size 4x4
```

## 赋值和改变大小

赋值操作是将一个矩阵拷贝到另一个矩阵的行为，使用=操作符。Eigen会自动改变左边的矩阵大小，和=右边的大小保持一致。

```c++
MatrixXf a(2,2);
std::cout << "a is of size " << a.rows() << "x" << a.cols() << std::endl;
MatrixXf b(3,3);
a = b;
std::cout << "a is now of size " << a.rows() << "x" << a.cols() << std::endl;

// output
a is of size 2x2
a is now of size 3x3
```

当然，如果左边是固定大小的矩阵，那么改变大小是不被允许的。

如果你不希望这种情况自动发生，那么你可以禁用它。（TODO）。

## 固定大小和动态大小的比较

什么时候要用固定大小的矩阵，如Matrix4f，以及什么时候更加倾向于动态矩阵，如MatrixXf。简单的答案是：针对非常小的矩阵使用固定大小的，对于大的矩阵使用动态大小的。对于小的矩阵，尤其是小于16（粗略的估计），使用固定大小的矩阵，对于性能更为有利，因为这样能够避免内存的分配以及释放等。在内部，固定大小的矩阵就是一个数组。如`Matrix4f mymatrix;`的行为和`float mymatrix[16];`的行为基本上是一致的。

作为比较，动态大小的矩阵，内存空间是分配再堆上面的，如`MatrixXf mymatrix(rows,columns);`和`float *mymatrix = new float[rows*columns];`比较类似的，不同的是前者存储了rows和columns作为成员变量。

使用固定大小的限制是，在编译阶段你必须要知道矩阵的大小。对于大的矩阵，使用固定大小带来的性能的提升可以被忽略不计。更糟糕的是，试图创建一个非常大的固定大小的矩阵，可能会导致栈溢出，因为Eigen将分配得到的数组作为局部变量，而局部变量存在于栈上。最后，和环境相关的是，在使用动态大小的时候，Eigen会尽可能的使用向量化（使用SIMD指令）。

## 可选的模板参数

在这一页开头，介绍了Matrix有六个模板参数，但是到目前为止，我们仅仅讨论了3个，剩下的参数如下：

```c++
Matrix<typename Scalar,
       int RowsAtCompileTime,
       int ColsAtCompileTime,
       int Options=0,
       int MaxRowsAtCompileTime = RowsAtCompileTime,
       int MaxColsAtCompileTime = ColsAtCompileTime>
```

- Options: 一个位字段，这里讨论一个呗，RowMajor。用来定义矩阵内部存储的是行为主的。默认情况下是列为主的。可以这么使用`Matrix<float,3,3,RowMajor>`
- MaxRowsAtCompileTime和MaxColsAtCompileTime，使用这两个参数的最大个一个原因是，你需要避免内存的动态分配，举个例子，下面的矩阵使用了12个float的数组，但没有动态分配内存空间。`Matrix<float,Dynamic,Dynamic,0,3,4>`

## 方便的typedefs

Eigen中定义了如下的typedefs：

- MatrixNt的格式用于定义`Matrix<type,N,N>`.如：MatrixXi为`Matrix<int,Dynamic,Dynamic>`
- VectorNt的格式用于定义`Matrix<type,N,1>`.如：Vector2f为`Matrix<float,2,3>`
- RowVectorNt的格式用于定义`Matrix<type,1,N>`.如：RowVector3d为`Matrix<double,1,3>`

上面的格式中，

- N可以是2,3,4，X中的任意一个
- t可以是i(int), f(float), d(double), cf(complex<float>), cd(complex<double>)中的任意一个。这里给出了5个标量类型，但不是说明，一定得这几个，可以进行扩展。
