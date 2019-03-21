# 快速参看指南

## 模块和头文件

Eigen库包括核心模块和几个附加模块。每个模块都有相关的头文件，使用这些模块，
就需要include对应的头文件。通过Eigen的头文件可以很方便的使用模块。

| 模块 | 头文件 | 内容 |
|------|--------|------|
| Core | `#include <Eigen/Core>` | 矩阵和数组类，基本的线性代数，数组操作 |
| Geometry | `#include <Eigen/Geometry>` | 包括形变，平移，缩放，旋转 |
| LU | `#include <Eigen/LU>` | 求逆，行列式，求解器进行LU分解 |
| Cholesky | `#include <Eigen/Cholesky>` | LLT LDLT Cholesky分解 |
| Householder | `#include <Eigen/Householder>` | Householder矩阵变换，这个模块被其他线性代数模块使用 |
| SVD | `#include <Eigen/SVD>` | 使用最小均方求解器进行SVD分解 |
| QR | `#include <Eigen/QR>` | QR分解 |
| Eigenvalues | `#include <Eigen/Eigenvalues>` | eigenvalue, eigenvector分解 |
| Sparse | `#include <Eigen/Sparse>` | 稀疏矩阵存储，相关的线性代数 |
|  | `#include <Eigen/Dense>` | 包括Core，Geometry，LU，Cholesky，SVD，QR，Eigenvalues的头文件 |
|  | `#include <Eigen/Eigen>` | 包括Sparse和Dense，整个Eigen库 |

## 数组，矩阵，向量

回忆下，Eigen提供了两种dense对象，矩阵和向量。都用Matrix类模板实现；
通用的1维和2维数组用Array末班类表示：

```c++
typedef Matrix<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options> MyMatrixType;
typedef Array<Scalar, RowsAtCompileTime, ColsAtCompileTime, Options> MyArrayType;
```

- Scalar是各个元素的标量类型（如，float，double，bool，int等）
- RowsAtCompileTime和 ColsAtCompileTime用来在编译时间确定矩阵的行列大小
- Options可以用来标注矩阵中元素存储是行为主，还是列为主的。

各种各样的组合都是合理的，你可以使用行固定大小，列动态大小的矩阵等。
下面的实现都是合理的：

```c++
Matrix<double, 6, Dynamic>                   // 列数目动态改变
Matrix<double, Dynamic, 2>                   // 行数目动态改变
Matrix<double, Dynamic, Dynamic, RowMajor>   // 都是动态改变的，行为主的存储方式
Matrix<double, 6, 3>                         // 行列数目都是固定大小
```

大多数情况下，可以使用预先定义好的矩阵和数组的typedef形式。一些例子为：

```c++
// matrices
Matrix<float,Dynamic,Dynamic>    <=>  MatrixXf
Matrix<double,Dynamic,1>         <=>  VectorXd
Matrix<int,1,Dynamic>            <=>  RowVectorXi
Matrix<float,3,3>                <=>  Matrix3f
Matirx<float,4,1>                <=>  Vector4f

// Arrays
Array<float,Dynamic,Dynamic>    <=>  ArrayXXf
Array<double,Dynamic,1>         <=>  ArrayXd
Array<int,1,Dynamic>            <=>  RowArrayXi
Array<float,3,3>                <=>  Array33f
Array<float,4,1>                <=>  Array4f
```

矩阵和数组之间的转换：

```c++
Array44f a1, a2;
Matrix4f m1, m2;
m1 = a1 * a2;                      // coefficient乘法，隐式将array转换成matrix
a1 = m1 * m2;                      // 矩阵乘法 
a2 = a1 + m1.array();              // 将矩阵和数组混合使用是禁止的
m2 = a1.matrix() + m1;             // 显示的转换是必须的
ArrayWrapper<Matrix4f> m1a(m1);    // m1a和m1，他们共享相同的内容
MatrixWrapper<Array44f> a1m(a1);
```

### 基本的矩阵操作

```c++
// 1. 构造
// 注意：默认情况下元素是没有初始化的
/// 1D对象
Vector4d v4;
Vector2f v1(x,y);
Array3i  v2(x,y,z);
Vector4d v3(x,y,z,w);

VectorXf v5;
ArrayXf  v6(size);

/// 2D对象
Matrix4f m1;
MatrixXf m5;
MatrixXf m6(nb_rows, nb_columns);

// 2. 逗号初始化
/// 1D对象
Vector3f v1; v1 << x,y,z;
ArrayXf v2(4); v2 << 1,2,3,4;

/// 2D对象
Matrix3f m1; m1 << 1,2,3,
                   4,5,6,
                   7,8,9;

int rows=5, cols=5;
MatrixXf m(rows,cols);
m << (Matrix3f() << 1,2,3,4,5,6,7,8,9).finished(),
     MatrixXf::Zero(3,cols-3),
     MatrixXf::Zero(rows-3,3),
     MatrixXf::Identity(rows-3,cols-3);

// 3. 运行时信息
// 注意：inner和outer与存储顺序相关
/// 1D对象
vector.size();
vector.innerStride();
vector.data();

/// 2D对象
matrix.rows();          matrix.cols();
matrix.innerSize();     matrix.outerSize();
matrix.innerStride();   matrix.outerStride();
matrix.data();

// 4. 编译时信息
ObjectType::Scalar            ObjectType::RowsAtCompileTime 
ObjectType::RealScalar        ObjectType::ColsAtCompileTime 
ObjectType::Index             ObjectType::SizeAtCompileTime 

// 5. 改变大小
/// 1D对象
vector.resize(size);
vector.resizeLike(other_vector);
vector.conservativeResize(size);

/// 2D对象
matrix.resize(nb_rows,nb_cols);
matrix.resize(Eigen::NoChange,nb_cols);
matrix.resize(nb_rows,Eigen::NoChange);
matrix.resizeLike(other_matrix);
matrix.conservativeResize(nb_rows,nb_cols);

// 6. 元素访问与范围check
// 注意：如果编译的时候定义了NDEBUG或EIGEN_NO_DEBUG
//       那么不会进行范围check
vector(i)     vector.x()
vector[i]     vector.y()
              vector.z()
              vector.w()
matrix(i,j)

// 7. 不需要范围check的元素访问
vector.coeff(i)
vector.coeffRef(i)
matrix.coeff(i,j)
matrix.coeffRef(i,j)

// 8. 赋值和拷贝
object = expression;
object_of_float = expression_of_double.cast<float>();
```

### 预定义的矩阵

```c++
// 固定大小的矩阵或向量
typedef {Matrix3f|Array33f} FixedXD;
FixedXD x;

x = FixedXD::Zero();
x = FixedXD::Ones();
x = FixedXD::Constant(value);
x = FixedXD::Random();
x = FixedXD::LinSpaced(size, low, high);
x.setZero();
x.setOnes();
x.setConstant(value);
x.setRandom();
x.setLinSpaced(size, low, high);

x = FixedXD::Identity();
x.setIdentity();

Vector3f::UintX();
Vector3f::UintY();
Vector3f::UintZ();

// 动态大小矩阵
typedef {Matrixf|ArrayXXf} Dynamic2D;
Dynamic2D x;

x = Dynamic2D::Zero(rows, cols);
x = Dynamic2D::Ones(rows, cols);
x = Dynamic2D::Constant(rows, cols, value);
x = Dynamic2D::Random(rows, cols);
x.setZero(rows, cols);
x.setOnes(rows, cols);
x.setConstant(rows, cols, value);
x.setRandom(rows, cols);

x = Dynamic2D::Identity(rows, cols);
x.setIdentity(rows, cols);

// 动态大小的向量
typedef {VectorXf|ArrayXf} Dynamic1D;
Dynamic1D x;
x = Dynamic1D::Zero(size);
x = Dynamic1D::Ones(size);
x = Dynamic1D::Constant(size, value);
x = Dynamic1D::Random(size);
x = Dynamic1D::LinSpaced(size, low, high);
x.setZero(size);
x.setOnes(size);
x.setConstant(size, value);
x.setRandom(size);
x.setLinSpaced(size, low, high);

vectorXf::Unit(size,i);
VectorXf::Unit(4,1) == Vector4f(0,1,0,0) == Vector4f::UnitY()
```

### 外部数组类型到Eigen类型的映射

```c++
// 具有连续内存空间
float data[] = {1,2,3,4};
Map<Vector3f> v1(data);
Map<ArrayXf> v2(data,3);
Map<Array22f> m1(data);
Map<MatrixXf> m2(data,2,2);

// stride间隔的使用
float data[] = {1,2,3,4,5,6,7,8,9};
Map<VectorXf,0,InnerStride<2>> v1(data,3);                    // v = [1,3,5]
Map<VectorXf,0,InnerStride<>>  v2(data,3,InnerStride<>(3));   // = [1,4,7]
Map<MatrixXf,0,OuterStride<3>> m2(data,2,3);                  // 两个结果都是 [1,4,7]
Map<MatrixXf,0,OuterStride<>>  m1(data,2,3,OuterStride<>(3)); //              [2,5,8]
```

## 算术操作

```c++
// 加减
mat3 = mat1 + mat2;           mat3 += mat1;
mat3 = mat1 - mat2;           mat3 -= mat1;

// 标量操作
mat3 = mat1 * s1;             mat3 *= s1;           mat3 = s1 * mat1;
mat3 = mat1 / s1;             mat3 /= s1;

// 矩阵/向量乘法*
col2 = mat1 * col1;
row2 = row1 * mat1;           row1 *= mat1;
mat3 = mat1 * mat2;           mat3 *= mat1;

// 转置/共厄矩阵
mat1 = mat2.transpose();      mat1.transposeInPlace();
mat1 = mat2.adjoint();        mat1.adjointInPlace();

// 点积/内积
scalar = vec1.dot(vec2);
scalar = col1.adjoint() * col2;
scalar = (col1.adjoint() * col2).value();

// 外积
mat = col1 * col2.transpose();

// 范式/归一化
scalar = vec1.norm();     scalar = vec1.squareNorm();
vec2 = vec1.normalized(); vec1.normalize();

// 叉乘
#include <Eigen/Geometry>
vec3 = vec1.cross(vec2);
```

## coefficient-wise及数组操作

除了上面提到的操作之外，Eigen还支持大量的coefficient-wise操作和函数。
大多数明确的指出，针对数组才是有意义的。下面的操作适用于数组，以及
通过array()接口调用的矩阵和向量。

```c++
// 算术操作
array1 * array2     array1 / array2     array1 *= array2    array1 /= array2
array1 + scalar     array1 - scalar     array1 += scalar    array1 -= scalar

// 比较操作
array1 < array2     array1 > array2     array1 < scalar     array1 > scalar
array1 <= array2    array1 >= array2    array1 <= scalar    array1 >= scalar
array1 == array2    array1 != array2    array1 == scalar    array1 != scalar
array1.min(array2)  array1.max(array2)  array1.min(scalar)  array1.max(scalar)

// 立方，平方，三角函数等
array1.abs2()
array1.abs()                  abs(array1)
array1.sqrt()                 sqrt(array1)
array1.log()                  log(array1)
array1.log10()                log10(array1)
array1.exp()                  exp(array1)
array1.pow(array2)            pow(array1,array2)
array1.pow(scalar)            pow(array1,scalar)
                              pow(scalar,array2)
array1.square()
array1.cube()
array1.inverse()
array1.sin()                  sin(array1)
array1.cos()                  cos(array1)
array1.tan()                  tan(array1)
array1.asin()                 asin(array1)
array1.acos()                 acos(array1)
array1.atan()                 atan(array1)
array1.sinh()                 sinh(array1)
array1.cosh()                 cosh(array1)
array1.tanh()                 tanh(array1)
array1.arg()                  arg(array1)
array1.floor()                floor(array1)
array1.ceil()                 ceil(array1)
array1.round()                round(aray1)
array1.isFinite()             isfinite(array1)
array1.isInf()                isinf(array1)
array1.isNaN()                isnan(array1)
```

下面的操作适用于所有的表达式（矩阵、向量、数组）

```c++
mat1.real();  real(array1);
mat1.imag();  imag(array1);
mat1.conjugate(); conj(array1);
```

对于矩阵和向量，有一些可以通过cwise方法实现。

``` 
MatrixAPI                                     通过数组转换
mat1.cwiseMin(mat2) mat1.cwiseMin(scalar)     mat1.array().min(mat2.array())  mat1.array().min(scalar)
mat1.cwiseMax(mat2) mat1.cwiseMax(scalar)     mat1.array().max(mat2.array())  mat1.array().max(scalar)
mat1.cwiseAbs2()                              mat1.array().abs2()
mat1.cwiseAbs()                               mat1.array().abs()
mat1.cwiseSqrt()                              mat1.array().sqrt()
mat1.cwiseInverse()                           mat1.array().inverse()
mat1.cwiseProduct(mat2)                       mat1.array() * mat2.array()
mat1.cwiseQuotient(mat2)                      mat1.array() / mat2.array()
mat1.cwiseEqual(mat2) mat1.cwiseEqual(scalar) mat1.array() == mat2.array()  mat1.array() == scalar 
mat1.cwiseNotEqual(mat2)                      mat1.array() != mat2.array()
```

上述两个api之间的主要区别是，cwise方法，返回的是一个矩阵世界的表达式，
而array方法返回的是数组表达式。array方法，并没有什么开销，知识改变了数据解析方式。

也可以很方便的使用用户定义函数

```c++
mat1.unaryExpr(std::ptr_fun(foo));
mat1.unaryExpr(std::ref(foo));
mat1.unaryExpr([](double x) {return foo(x);});
```

## Reductions

Eigen提供了一些reductions的函数，如：`minCoeff()`, `maxCoeff()`, `sum()`, `prod()`, `trace()`, `norm()`,
`squaredNorm()`,`all()`,`any()`。所有的reduction函数都可以基于矩阵，基于行或基于列进行操作。

```c++
MatrixXf mat(3,3);
mat << 5,3,1,2,7,8,9,4,6;

mat.minCoeff(); // 1 
mat.colwise().minCoeff(); // 2 3 1 
mat.rowwise().minCoeff(); // [1;2;4]
```

minCoeff和maxCoeff特殊的用法：

```c++
int i,j;
s = vector.minCoeff(&i);   // s == vector[i]
s = matrix.maxCoeff(&i,&j); // s == matrix(i,j)
```

all和any的用法：

```c++
if ((array1 > 0).all()) { ... } // 如果array1中所有元素都大于0
if ((array1 < array2).any()) { ... } // 如果有一个对应的元素满足array1(i,j) < array2(i,j)
```

## 子矩阵

读写访问矩阵中的一行或一列：

```c++
mat1.row(i) = mat2.col(j);
mat1.col(j1).swap(mat1.col(j2));
```

子向量的读写访问：

```
默认的版本            编译时大小确定
vec1.head(n)          vec1.head<n>()              获取前n个元素
vec1.tail(n)          vec1.tail<n>()              获取最后n个元素
vec1.segment(pos,n)   vec1.segment<n>(pos)        从位置p开始获取n个元素
```

子矩阵的读写访问：

``` 
mat1.block(i,j,rows,cols)                    mat1.block<rows,cols>(i,j)
mat1.topLeftCorner(rows,cols)                mat1.topLeftCorner<rows,cols>()
mat1.topRightCorner(rows,cols)               mat1.topRightCorner<rows,cols>()
mat1.bottomLeftCorner(rows,cols)             mat1.bottomLeftCorner<rows,cols>()
mat1.bottomRightCorner(rows,cols)            mat1.bottomRightCorner<rows,cols>()

mat1.topRows(rows)                           mat1.topRows<rows>()
mat1.bottomRows(rows)                        mat1.bottomRows<rows>()
mat1.leftCols(cols)                          mat1.leftCols<cols>()
mat1.rightCols(cols)                         mat1.rightCols<cols>()
```

## 其他操作

### 反转

向量，矩阵的行列都可以反转：

```c++
vec.reverse(); mat.colwise().reverse(); mat.rowwise().reverse();
vec.reverseInPlace();
```

### 重复

向量，矩阵的行列都可以进行照着特定方向重复。

```c++
vec.replicate(times);
vec.replicate<Times>();

mat.replicate(vertical_times, horizontal_times);
mat.replicate<VerticalTimes, HorizontalTimes>();

mat.colwise().replicate(vertical_times, horizontal_times);
mat.colwise().replicate<VerticalTimes, HorizontalTimes>();

mat.rowwise().replicate(vertical_times, horizontal_times);
mat.rowwise().replicate<VerticalTimes, HorizontalTimes>();
```

## 对角矩阵，三角矩阵和自共厄矩阵

### 对角矩阵

```c++
mat1 = vec1.asDiagnoal();     // 向量转换成对角矩阵
DiagonalMatrix<Scalar, SizeAtCompileTime> diag1(size);  // 声明对角矩阵
diag1.diagonal() = vector;

// 访问对角矩阵
vec1 = mat1.diagonal();
mat1.diagonal() = vec1;    // 主对角元素

vec1 = mat1.diagonal(+n);  mat1.diagonal(+n) = vec1; // 上对角矩阵，第0行，第n列，开始的对角矩阵
vec1 = mat1.diagonal(-n);  mat1.diagonal(-n) = vec1; // 刚好与上面相反

vec1 = mat1.diagonal<1>(); mat.diagonal<1>() = vec1; // 含义于上面相同
vec1 = mat1.diagonal<-2>(); mat.diagonal<-2>() = vec1; 

// 相乘与求逆
mat3 = scalar * diag1 * mat1;
mat3 += scalar * mat1 * vec1.asDiagnoal();
mat3 = vec1.asDiagnoal().inverse() * mat1;
mat3 = mat1 * diag1.inverse();
```

### 三角矩阵

TriangularView用来表示稠密矩阵中的三角部分，用来执行一些优化操作。另一半的三角，
将不再被使用，也无法用来存储其他额外的信息。

> 注意：
> 如果triangularView模板成员函数被用成一个带有模板参数的对象的时候，需要template关键字

```c++
// 用来表示特定形式的矩阵
m.triangularView<Xxx>(); // Xxx = Upper, Lower, StrictlyUpper, StrictlyLower, UnitUpper, UnitLower

// 写入特定的部分
m1.triangularView<Eigen::Lower>() = m2 + m3;

// 将稠密矩阵的相对的一个三角部分赋值位0
m2 = m1.triangularView<Eigen::UnitUpper>();

// 乘法
m3 += s1 * m1.adjoint().triangularView<Eigen::UnitUpper>() * m2;
m3 -= s1 * m2.conjugate() * m1.adjoint().triangularView<Eigen::Lower>();

// 解线性方程
L1.triangularView<Eigen::UintLower>().solveInPlace(M2);              // M2 = L1^-1 * M2
L1.triangularView<Eigen::Lower>().adjoint().solveInPlace(M3);        // M3 = L1^(*-1) * M3 
U1.triangularView<Eigen::Upper>().soleInPlace<OnTheRight>(M4);       // M4 = M4 * U^-1
```

### 对称/自共厄

正如三角矩阵，你可以将三角矩阵进行共厄，得到自共厄矩阵。同样的，三角矩阵的一般是不会被使用到的。

> 注意：
> selfadjiontView()模板函数使用的时候带有模板参数的化也需要template关键字。

```c++
// 转换成稠密矩阵
m2 = m.selfadjiontView<Eigen::Lower>();

// 与普通矩阵或向量相乘
m3 = s1 * m1.conjugate().selfadjiontView<Eigen::Upper>() * m3;
m3 -= s1 * m3.adjoint() * m1.selfadjointView<Eigen::Lower>();

M1.selfadjointView<Eigen::Upper>().rankUpdate(M2,s1); // upper(M1) += s1M2M2^{*}
M1.selfadjointView<Eigen::Lower>().rankUpdate(M2.adjoint(),-1); // lower(M1) -= M2^{*}*M2

M.selfadjointView<Eigen::Upper>().rankUpdate(u,v,s); // rank2: M += suv* + svu*

m2 = m1.selfadjointView<Eigen::Upper>().llt().solve(m2); // 标准cholesky因子分解，求解线性方程 M2 = M1^{-1} M2
m2 = m1.selfadjointView<Eigen::Lower>().ldlt().solve(m2); // 使用pivoting 的 Cholesky 因子分解
```


## 分解详解

### LU分解

- 什么是LU分解
- 为什么需要LU分解
- 怎么进行LU分解

可以参考： https://www.cnblogs.com/bigmonkey/p/9555710.html

