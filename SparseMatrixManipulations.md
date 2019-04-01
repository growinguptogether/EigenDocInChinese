# 稀疏矩阵操作
操作和求解稀疏矩阵问题涉及到的模块，总结如下：

|模块|头文件|包含内容|
|:----:|:----:|:----:|
|[SparseCore]|`#include <Eigen/SparseCore>`|[稀疏矩阵]和[稀疏向量]类,矩阵集合，基础稀疏线性代数（包括稀疏三角求解器）|
|[SparseCholesky]|`#include <Eigen/SparseCholesky>`|介绍了用于求解自共轭(self-adjoint)正定问题的稀疏[LLT]和[LDLT]Cholesky因数分解求解器|
|[SparseLU]|`#include <Eigen/SparseLU>`|求解通用稀疏方阵系统的LU因式分解求解器|
|[SparseQR]|`#include <Eigen/SparseQR>`|求解稀疏线性近方(least-squares)问题的QR因式分解求解器|
|[IterativeLinearSolvers]|`#include <Eigen/IterativeLinearSolvers>`|求解大型通用线性方阵问题（包括自共轭正定问题）的迭代求解器|
|[Sparse]|`#include <Eigen/Sparse>`|包括以上所有模块|

# 稀疏矩阵格式

有很多场景(例如，有限元方法)，我们需要处理非常大同时只有少量因数非零的矩阵。在这种场景下，我们可以通过特殊表示法来保存那些非零因数来减少内存开销同时加快程序执行速度。这种矩阵，我们称之为稀疏矩阵。

**稀疏矩阵类**

