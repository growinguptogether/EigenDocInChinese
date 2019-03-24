# 稀疏线性系统求解

在Eigen中，有多种方法可以求解稀疏矩阵稀疏的线性系统。由于这类矩阵表达的特殊性，需要重点关注求解方法的性能问题。详细了解Eigen中的系数矩阵，可查看[稀疏矩阵操作]一节。该页面列举了Eigen中的稀疏求解器。其主要步骤与介绍过的线性求解器一致。根据稀疏矩阵的属性，即期望精度，最终用户能够调整这些步骤来提高代码的性能。注意：没有必要去深度了解这些步骤背后隐藏的内容；上一节介绍了一个基准程序，可以很容易地获取求解器的性能。

### 稀疏求解器列表
Eigen目前提供了一系列的内置求解器以及外部求解器库的包装器，总结如下表：
类名(头文件)|求解器类型|矩阵类型|性能相关特征|协议|备注
-|-|-|-|-|-
SimplicialLLT<br>(#include<Eigen/SparseCholesky>)|LLT分解法|对称正定矩阵|最小化填充矩阵降低消耗内存和算术运算|LGPL|SimplicialLDLT更优
SimplicialLDLT<br>(#include<Eigen/SparseCholesky>)|LDLT分解法|对称正定矩阵|最小化填充矩阵降低消耗内存和算术运算|LGPL|建议用于矩阵非常稀疏且不太大的问题(如二维泊松方程求解)
SparseLU<br>(#include<Eigen/SparseLU>)|LU分解|方阵|矩阵最小化填充、快速密集迭代|MPL2|针对不规则格式的小型和大型矩阵求解问题进行了优化
SparseQR<br>(#include<Eigen/SparseQR>)|QR分解任何矩形阵|矩阵最小化填充|MPL2|建议用于最小二乘问题，具有基本的秩显功能

### 内置迭代求解器
类名(头文件)|求解器类型|矩阵类型|支持的预处理器[默认]|协议|备注
-|-|-|-|-|-
ConjugateGradient<br>(#include<Eigen/IterativeLinearSolvers>)|经典迭代CG算法求解|对称正定矩阵|	IdentityPreconditioner, [DiagonalPreconditioner], IncompleteCholesky|MPL2|建议用于大型对称矩阵求解(如3D泊松方程求解)
LeastSquaresConjugateGradient<br>(#include<Eigen/IterativeLinearSolvers>)|矩形最小二乘问题CG求解|矩形阵|IdentityPreconditioner, [LeastSquareDiagonalPreconditioner]|MPL2|在没有形成A'A的情况下求解min(A'Ax-b)^2
BiCGSTAB<br>(#include<Eigen/IterativeLinearSolvers>)|双共轭梯度稳定迭代|方阵|IdentityPreconditioner, [DiagonalPreconditioner], IncompleteLUT|MPL2|要加速收敛，请使用IncompleteLUT预处理器进行尝试

### 外部求解器封装
类名|模块|矩阵类型|性能相关特征|依赖、协议|备注
-|-|-|-|-|-