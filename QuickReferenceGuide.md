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



## 分解详解

### LU分解

- 什么是LU分解
- 为什么需要LU分解
- 怎么进行LU分解

可以参考： https://www.cnblogs.com/bigmonkey/p/9555710.html

