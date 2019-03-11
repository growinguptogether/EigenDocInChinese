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

## 基本的矩阵操作

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

## 分解详解

### LU分解

- 什么是LU分解
- 为什么需要LU分解
- 怎么进行LU分解

可以参考： https://www.cnblogs.com/bigmonkey/p/9555710.html

