# 稀疏矩阵操作
操作和求解稀疏矩阵问题涉及到的模块，总结如下：

|模块|头文件|包含内容|
|:----:|:----|:----:|
|[SparseCore]|`#include <Eigen/SparseCore>`|[稀疏矩阵][sm]和[稀疏向量](http://eigen.tuxfamily.org/dox/classEigen_1_1SparseVector.html)类,矩阵集合，基础稀疏线性代数（包括稀疏三角求解器）|
|[SparseCholesky]|`#include <Eigen/SparseCholesky>`|介绍了用于求解自共轭(self-adjoint)正定问题的稀疏[LLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LLT.html)和[LDLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LDLT.html)Cholesky因数分解求解器|
|[SparseLU]|`#include <Eigen/SparseLU>`|求解通用稀疏方阵系统的LU因式分解求解器|
|[SparseQR]|`#include <Eigen/SparseQR>`|求解稀疏线性最小二乘法(least-squares)问题的QR因式分解求解器|
|[IterativeLinearSolvers]|`#include <Eigen/IterativeLinearSolvers>`|求解大型通用线性方阵问题（包括自共轭正定问题）的迭代求解器|
|[Sparse]|`#include <Eigen/Sparse>`|包括以上所有模块|

# 稀疏矩阵格式

有很多场景(例如，有限元方法)，我们需要处理非常大同时只有少量因数非零的矩阵。在这种场景下，我们可以通过特殊表示法来保存那些非零因数来减少内存开销同时加快程序执行速度。这种矩阵，我们称之为稀疏矩阵。

**稀疏矩阵类**
[SparseMatrix][sm]类是[Eigen]稀疏模组中描述稀疏矩阵的类；该类同时具有高运算性能同时内存开销小的特点。它实现了一种比广泛使用的列（行）压缩存储更通用更灵活的存储策略。它由四个简单的数组组成：
- Values:储存非零因数。
- InnerIndices:储存每一行(列)非零因因数的索引值。
- OuterStarts:储存每一列(行)前两个数组中第一个非零因数的索引。
- InnerNNZs:储存每一列（行）中非零的因数。单词inner的意义是内部向量，对列主序矩阵是一列，对行主序矩阵是一行。单词outer指的是另一个方向。

用下面的例子可以更好的解释我们的策略。下面的矩阵：
```
0	3	0	0	0
22	0	0	0	17
7	5	0	1	0
0	0	0	0	0
0  14*	0*	0	8
```
*译者注：这里原文0和14是反过来的，根据后文，我认为原文给反了。
这是它的一种可能的稀疏**列主序**表示形式：
```
Values:	        22	7	_	3	5	14	_	_	1	_	17	8
InnerIndices:	1	2	_	0	2	4	_	_	2	_	1	4
OuterStarts:	0	3	5	8	10	12
InnerNNZs:	    2	2	1	1	2	
```
这里每个内部向量里元素的顺序是按其在内部向量索引值升序排列的。“_”表示用于快速插入新元素预留的空间。现在，假设不需要调整已分配内存空间大小，随机插入一个元素的复杂度为`O(nnz_j)`，这里nnz_j是

[SparseCore]:http://eigen.tuxfamily.org/dox/group__SparseCore__Module.html
[SparseCholesky]:http://eigen.tuxfamily.org/dox/group__SparseCholesky__Module.html
[SparseLU]:http://eigen.tuxfamily.org/dox/group__SparseLU__Module.html
[SparseQR]:http://eigen.tuxfamily.org/dox/group__SparseQR__Module.html
[IterativeLinearSolvers]:http://eigen.tuxfamily.org/dox/group__IterativeLinearSolvers__Module.html
[Sparse]:http://eigen.tuxfamily.org/dox/group__Sparse__Module.html
[sm]:http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrix.html
[Eigen]:http://eigen.tuxfamily.org/dox/namespaceEigen.html