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

## 分解详解

### LU分解

- 什么是LU分解
- 为什么需要LU分解
- 什么情况下使用LU分解
- 什么情况下不适合LU分解
- 怎么进行LU分解

下面分别来回答这几个问题。

#### 什么是LU分解？

LU分解就是将矩阵分解成一个下三角矩阵L和一个上三角矩阵U的乘积。

#### LU分解的优点是什么

https://math.stackexchange.com/questions/2010045/what-is-the-advantage-of-lu-factorization

#### 为什么需要LU分解？

1. LU分解能够简化方程组求解。
 
#### 怎么进行LU分解


