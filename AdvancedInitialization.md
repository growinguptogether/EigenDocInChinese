# 高阶初始化

> ref: http://eigen.tuxfamily.org/dox/group__TutorialAdvancedInitialization.html

这一页讨论了几个初始化矩阵的高阶方法。更加详细的介绍了之前提到过的逗号初始化。
同时，也说明了如何初始化特殊矩阵，如单位矩阵和零矩阵。

## 逗号初始化

为了更方便的对矩阵或数组或向量的所有元素进行初始化，Eigen提供了逗号初始化的方式。
只需要简单的将元素列出来，从左上角开始，从左到右，从上到下，依次列出。
对象的大小需要提前设定。如果提供的元素个数偏少，或偏多，Eigen都会抱怨。

```c++
Matrix3f m;
m << 1,2,3,
     4,5,6,
     7,8,9;
std::cout << m;

// output 
1 2 3 
4 5 6
7 8 9
```

此外，初始化列表中的元素可以是向量或是矩阵。这种常见的形式是将向量或矩阵连接起来。
比如，下面例子给出了如何将两个向量连接到一块。需要注意的是，使用逗号初始化之前，
必须设定大小。

```c++
RowVectorXd vec1(3);
vec1 << 1,2,3;
std::cout << "vec1 = " << vec1 << std::endl;

RowVectorXd vec2(4);
vec2 << 1,4,6,9;
std::cout << "vec2 = " << vec2 << std::endl;

RowVectorXd joined(7);
joined << vec1, vec2;
std::cout << "joined = " << joined << std::endl;

// output 
vec1 = 1 2 3
vec2 =  1  4  9 16
joined =  1  2  3  1  4  9 16
```

我们可以使用相同的技术来初始化矩阵。

```c++
MatrixXf matA(2,2);
matA << 1,2,3,4;
MatrixXf matB(4,4);
matB << matA, matA/10, matA/10, matA;
std::cout << matB << std::endl;

// output 
1   2 0.1 0.2
  3   4 0.3 0.4
0.1 0.2   1   2
0.3 0.4   3   4
```

逗号初始化的方式也可用来对块中的数据进行填充，如`m.row(i)`。
下面给出了相对于第一个例子更加复杂的实现方式：

```c++
Matrix3f m;
m.row(0) << 1,2,3;
m.block(1,0,2,2) << 4,5,7,8;
m.col(2).tail(2) << 6,9;
std::cout << m;

// output
1 2 3 
4 5 6
7 8 9
```

## 特殊的矩阵和向量

矩阵或数组有像`Zero()`的静态方法，用来将所有的元素初始化成零。这里有三种形态。
第一种不需要传递参数，仅适用于固定大小的对象。如果你需要对动态大小的对象初始化成0，
你需要设定大小。因此，第二种形态需要传入一个参数，适用于一维动态大小的对象。
第三种形态是需要传入两个参数，适用于二维对象。三种使用方式见下面的例子：

```c++
cout << "A fixed-size array:\n";
Array33f a1 = Array33f::Zero();
cout << a1 << "\n\n";

cout << "A one-dimensional dynamic-size array:\n";
ArrayXf a2 = ArrayXf::Zero(3);
cout << a2 << "\n\n";

cout << "A tow-dimensional dynamic-size array:\n";
ArrayXXf a3 = ArrayXXf::Zero(3,4);
cout << a3 << "\n";

// output 
A fixed-size array:
0 0 0
0 0 0
0 0 0

A one-dimensional dynamic-size array:
0
0
0

A two-dimensional dynamic-size array:
0 0 0 0
0 0 0 0
0 0 0 0
```

类似的，`Constant(value)`静态方法，将所有的元素设置成`value`。如果对象的大小需要给出，
那么需要传入额外的参数，如`MatrixXd::Constant(rows,cols,value)`。`Random()`函数会将矩阵或数组的元素随机赋值。
单位矩阵可以通过调用`Identity()`构建，这个方法仅适用于Matrix，不适用于Array，因为单位矩阵是线性代数的概念。
函数`LinSpaced(size,low,high)`仅适用于向量和和一维数组，用来产生`low`和`high`之间均匀分布的，大小为`size`的元素。
下面你的例子给出了`LinSpaced`的使用，来产生角度，弧度，以及对应的sine和cosine值。

```c++
ArrayXXf table(10, 4);
table.col(0) = ArrayXf::LinSpaced(10, 0, 90);
table.col(1) = M_PI / 180 * table.col(0);
table.col(2) = table.col(1).sin();
table.col(3) = table.col(1).cos();
std::cout << "  Degrees   Radians      Sine    Cosine\n";
std::cout << table << std::endl;

// output
Degrees   Radians      Sine    Cosine
        0         0         0         1
       10     0.175     0.174     0.985
       20     0.349     0.342      0.94
       30     0.524       0.5     0.866
       40     0.698     0.643     0.766
       50     0.873     0.766     0.643
       60      1.05     0.866       0.5
       70      1.22      0.94     0.342
       80       1.4     0.985     0.174
       90      1.57         1 -4.37e-08
```

这个例子展示了，LinSpaced()返回的对象可以赋值给变量（和表达式）。Eigen提供了更方便的函数来实现这个操作，
如，`setZero()`,`MatrixBase::setIdentity()`和`DenseBase::setLinSpaced()`。
下面的例子给出了构建下面矩阵的三种方式：静态函数加赋值，静态函数加逗号初始化，或者使用setXxx()函数。

![](http://latex.codecogs.com/gif.latex?J%20%3D%20%5Cbegin%7Bbmatrix%7D%20O%20%26%20I%5C%5C%20I%20%26%20O%20%5Cend%7Bbmatrix%7D)

```c++
const int size = 6;
MatrixXd mat1(size,size);
mat1.topLeftCorner(size/2,size/2)     = MatrixXd::Zero(size/2,size/2);
mat1.topRightCorner(size/2,size/2)    = MatrixXd::Identity(size/2,size/2);
mat1.bottomLeftCorner(size/2,size/2)  = MatrixXd::Identity(size/2,size/2);
mat1.bottomRightCorner(size/2,size/2) = MatrixXd::Zero(size/2,size/2);
cout << mat1 << endl << endl;

MatrixXd mat2(size,size);
mat2.topLeftCorner(size/2,size/2).setZero();
mat2.topRightCorner(size/2,size/2).setIdentity();
mat2.bottomLeftCorner(size/2,size/2).setIdentity();
mat2.bottomRightCorner(size/2,size/2).setZero();
cout << mat2 << endl << endl;

MatrixXd mat3(size,size);
mat3 << MatrixXd::Zero(size/2,size/2), MatrixXd::Identity(size/2,size/2),
        MatrixXd::Identity(size/2,size/2),MatrixXd::Zero(size/2,size/2);
cout << mat3 << endl;
```

所有预定义的矩阵，向量，数组对象的小结，可以在快速参考指导中找到。 TODO add link

## 使用临时对象

正如上面所述，`Zero()`和`Constant()`一类的静态函数可以在声明的时候使用，
也可以在赋值的时候作为等号右边的值。你可以想象这些方法返回了矩阵或是数组；
实际上它返回的是*表达式对象*，在需要的时候能够生成矩阵或数组，因此这些语法并不会产生多少开销。

这些表达式也可以被作为临时对象。[GettingStarted](./GettingStarted.md)介绍中已经有了对应的示例，下面再次提及：

```c++
#include <iostream>
#include <Eigen/Dense>

using namespace Eigen;
using namespace std;

int main()
{
  MatrixXd m = MatrixXd::Random(3,3);
  m = (m + MatrixXd::Constant(3,3,1.2)) * 50;
  cout << "m =" << endl << m << endl;
  VectorXd v(3);
  v << 1, 2, 3;
  cout << "m * v =" << endl << m*v << endl;
}

// output
m =
  94 89.8 43.5
49.4  101 86.8
88.3 29.8 37.8
m * v =
404
512
261
```

`m + MatrixXd::Constant(3,3,1.2)`表达式构建了3x3的各个元素都为1.2的矩阵，然后在和矩阵m做加法。

逗号初始化也可以被用作临时对象。如下例：

```c++
MatrixXf mat = MatrixXf::Random(2,3);
cout << mat << endl << endl;
mat = (MatrixXf(2,2) << 0, 1, 1, 0).finished() * mat;
cout << mat << endl;

// output
  0.68  0.566  0.823
-0.211  0.597 -0.605

-0.211  0.597 -0.605
  0.68  0.566  0.823
```

这里`finished()`方法是必要的，用来当初始化完成时生成对应的临时矩阵。
