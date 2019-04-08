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
- InnerIndices:储存按行(列)计算非零因因数的索引值。
- OuterStarts:储存按列(行)计算前两个数组中第一个非零因数的索引。
- InnerNNZs:储存每一列（行）中非零的因数。单词inner的意义是内部向量，对列主序矩阵是一列，对行主序矩阵是一行。单词outer指的是另一个方向。

用下面的例子可以更好的解释我们的策略。下面的矩阵：
```
0	3	0	0	0
22	0	0	0	17
7	5	0	1	0
0	0	0	0	0
0   0   14	0	8
```
这是它的一种可能的稀疏**列主序**表示形式：
```
Values:	        22	7	_	3	5	14	_	_	1	_	17	8
InnerIndices:	1	2	_	0	2	4	_	_	2	_	1	4
OuterStarts:	0	3	5	8	10	12
InnerNNZs:	    2	2	1	1	2	
```
这里每个内部向量里元素的顺序是按其在内部向量索引值升序排列的。“_”表示用于快速插入新元素预留的空间。现在，假设不需要调整已分配内存空间大小，随机插入一个元素的复杂度为`O(nnz_j)`，这里nnz_j是对应内向量里非零元素的个数。换而言之，按照内部向量索引升序的方式插入新元素执行效率会更高。因为这种操作只需要增加对应的`InnerNNZs`实例就可以了，这个操作复杂度为O(1)。
在存储时不预留额外空间的场景比较特殊，这种方式也被称之为*压缩存储*模式。它与广泛使用的列(行)压缩存储(CCS或CRS)策略是一致的。任意一个[稀疏矩阵][sm]都可以通过调用[SparseMatrix::makeCompressed()]方法将存储模式转换为压缩存储。此时，你可以看到`InnerNNZs`数组是由`OuterStarts`推断出来的，因为有等式`InnerNNZs[j] = OuterStarts[j+1]-OuterStarts[j]`存在。因此，在具体实现中，调用[SparseMatrix::makeCompressed()]方法通常会释放`InnerNNZs`的空间。

需要注意到，我们封装的三方库大多要求将压缩矩阵作为入参。
Eigen操作的返回值都是**压缩过**的稀疏矩阵。另一方面，向这种[稀疏矩阵][sm]插入新元素将导致该矩阵变成**非压缩**模式。
以下是压缩模式存储上面示例矩阵的说明：
```
Values:	        22	7	3	5	14	1	17	8
InnerIndices:	1	2	0	2	4	2	1	4
OuterStarts:	0	2	4	5	6	8
```
[稀疏向量][sv]是[稀疏矩阵][sm]只保存`Values`和`InnerIndices`数组的一种特殊情况。对于[稀疏向量][sv]并不存在压缩/非压缩模式的区别。

## 一个例子

在开始介绍每个具体的类之前，我们希望用下面这个典型的例子作为开始：在常规2D光栅内应用有限差分方法和狄利克雷边界条件求解拉普拉斯方程![](https://latex.codecogs.com/gif.download?%5Cfn_cm%20%5CDelta%20%5Cupsilon%20%3D%200)。这个问题在数学上可以表达为求解一个形如![](https://latex.codecogs.com/gif.download?%5Cfn_cm%20%5Cmathbf%7BA%7Dx%20%3D%20b)的线性方程，其中x是包含m个未知数的向量(我们例子里是像素的个数)，b是根据边界条件得到的右边向量，A是一个拉普拉斯算子离散化生成只有少数因数非0的`m x m`的矩阵。
```cpp
#include <Eigen/Sparse>
#include <vector>
#include <iostream>
typedef Eigen::SparseMatrix<double> SpMat; // declares a column-major sparse matrix type of double
typedef Eigen::Triplet<double> T;
void buildProblem(std::vector<T>& coefficients, Eigen::VectorXd& b, int n);
void saveAsBitmap(const Eigen::VectorXd& x, int n, const char* filename);
int main(int argc, char** argv)
{
  if(argc!=2) {
    std::cerr << "Error: expected one and only one argument.\n";
    return -1;
  }
  
  int n = 300;  // size of the image
  int m = n*n;  // number of unknows (=number of pixels)
  // Assembly:
  std::vector<T> coefficients;            // list of non-zeros coefficients
  Eigen::VectorXd b(m);                   // the right hand side-vector resulting from the constraints
  buildProblem(coefficients, b, n);
  SpMat A(m,m);
  A.setFromTriplets(coefficients.begin(), coefficients.end());
  // Solving:
  Eigen::SimplicialCholesky<SpMat> chol(A);  // performs a Cholesky factorization of A
  Eigen::VectorXd x = chol.solve(b);         // use the factorization to solve for the given right hand side
  // Export the result to a file:
  saveAsBitmap(x, n, argv[1]);
  return 0;
}
```
在这个例子中，我们先是定义了一个双精度列主序的稀疏矩阵`SparseMatrix<double>`,然后是一个同样类型的三元组列表`Triplet<double>`。`Triplet`是一种用三元组描述非零元素实例的一种简单对象，同时记录：行索引，列索引与元素值。

在主函数里，我们声明了一列三元组因数和等式右侧的向量b，然后用*buildProblem*方法对它们进行填充。原始扁平的非零元素表在这种方法作用下，真正转化成一个[稀疏矩阵][sm]对象A。请注意列表里的元素不再需要保留，可能重复的元素会被叠加起来。

最后一步是这个问题集合的高效求解部分。因为我们构造的结果矩阵A是堆成的，我们可以使用[SimplicialLDLT](http://eigen.tuxfamily.org/dox/classEigen_1_1SimplicialLDLT.html)类直接计算Cholesky因式分解，它的效率跟[LDLT]处理稠密对象类似。

结果向量x以一维数组的形式容纳了范例代码右侧jpeg图像（图像丢失了）里的像素值。

*buildProblem*和*save*方法的解释不在这个教程的范围内。如果你有兴趣或者希望自定义修改，它们的说明在[这里](http://eigen.tuxfamily.org/dox/TutorialSparse_example_details.html)。

## 稀疏矩阵类
**矩阵和向量的属性**
[稀疏矩阵][sm]和[稀疏向量][sv]类接受三个模板参数：标量类型（例如double），存储顺序（列优先还是行优先，默认是列优先）以及内部索引类型（默认是int）。
对于稠密[矩阵](http://eigen.tuxfamily.org/dox/classEigen_1_1Matrix.html)对象，构造函数直接申请对象全尺寸的内存。下面是例子：
```cpp
SparseMatrix<std::complex<float> > mat(1000,2000);         // declares a 1000x2000 column-major compressed sparse matrix of complex<float>
SparseMatrix<double,RowMajor> mat(1000,2000);              // declares a 1000x2000 row-major compressed sparse matrix of double
SparseVector<std::complex<float> > vec(1000);              // declares a column sparse vector of complex<float> of size 1000
SparseVector<double,RowMajor> vec(1000);                   // declares a row sparse vector of double of size 1000
```
在下面的教程中，*mat*和*vec*分别代表稀疏矩阵和稀疏向量对象。

矩阵的维度可以使用一下方法获取：

|-------|-------------------------------------------|
|标准维度|`mat.rows()`   `vec.size()`    `mat.cols()`|
|内部/外部方向的尺寸|`mat.innerSize()`   `mat.outerSize()`|
|非0因数的个数|`mat.nonZeros()`   `vec.nonZeros()`|
**迭代非零因数**
随机访问稀疏矩阵元素可以使用方法`coeffRef(i,j)`实现。但是这个方法使用了非常低效的二分查询法。大多数情况下，人们希望仅迭代那些非零的因数。这可以通过首先循环访问外部维度，然后对每个外部维度利用内部向量迭代器迭代其内部向量里的非零因素实现。如此，非零元素将按照其在矩阵中存储的顺序依次访问。下面是示例代码：
```cpp
SparseMatrix<double> mat(rows,cols);
for (int k=0; k<mat.outerSize(); ++k)
  for (SparseMatrix<double>::InnerIterator it(mat,k); it; ++it)
  {
    it.value();
    it.row();   // row index
    it.col();   // col index (here it is equal to k)
    it.index(); // inner index, here it is equal to it.row()
  }

SparseVector<double> vec(size);
for (SparseVector<double>::InnerIterator it(vec); it; ++it)
{
  it.value(); // == vec[ it.index() ]
  it.index();
}
```
对于一个可写表达式，引用传递的参数可以使用valueRef()方法修改其内容。如果稀疏矩阵或向量依赖于一个模板参数，你需要使用*typedef*关键字来指明InnerIterator表示一种类型；详细内容请参看[c++里的模板和类型关键字](http://eigen.tuxfamily.org/dox/TopicTemplateKeyword.html)

## 填充稀疏矩阵
因为[稀疏矩阵][sm]的存储采用了特殊策略，在向其添加新的非零因数时需要一些特殊操作。例如，对[稀疏矩阵][sm]进行依次完全随机插入的时间复杂度为O(nnz)，其中nnz是当前非零因数的个数。

构造一个高效的稀疏矩阵最简单的方法是先构造一个所谓的三元组列表，然后将它转换为[稀疏矩阵][sm]。

下面是典型的使用范例：
```cpp
typedef Eigen::Triplet<double> T;
std::vector<T> tripletList;
tripletList.reserve(estimation_of_entries);
for(...)
{
  // ...
  tripletList.push_back(T(i,j,v_ij));
}
SparseMatrixType mat(rows,cols);
mat.setFromTriplets(tripletList.begin(), tripletList.end());
// mat is ready to go!
```

[SparseCore]:http://eigen.tuxfamily.org/dox/group__SparseCore__Module.html
[SparseCholesky]:http://eigen.tuxfamily.org/dox/group__SparseCholesky__Module.html
[SparseLU]:http://eigen.tuxfamily.org/dox/group__SparseLU__Module.html
[SparseQR]:http://eigen.tuxfamily.org/dox/group__SparseQR__Module.html
[IterativeLinearSolvers]:http://eigen.tuxfamily.org/dox/group__IterativeLinearSolvers__Module.html
[Sparse]:http://eigen.tuxfamily.org/dox/group__Sparse__Module.html
[sm]:http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrix.html
[sv]:http://eigen.tuxfamily.org/dox/classEigen_1_1SparseVector.html
[Eigen]:http://eigen.tuxfamily.org/dox/namespaceEigen.html
[SparseMatrix::makeCompressed()]:http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrix.html#a5ff54ffc10296f9466dc81fa888733fd