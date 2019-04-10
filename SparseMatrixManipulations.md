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
|  类型 |                 方法                      |
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
容纳三元组的`std::vector`存储矩阵元素的顺序可能是乱序的，也有可能包含有将要用setFromTriplets()方法重叠的重复元素。这部分详细内容请参看[SparseMatrix::setFromTriplets()](http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrix.html#acc35051d698e3973f1de5b9b78dbe345)方法和[Triplet](http://eigen.tuxfamily.org/dox/classEigen_1_1Triplet.html)类。

然而，在某些场景下，更高的运行效率和更低的内存开销反而是通过直接向目标矩阵插入非零元素实现的。下面是这种场景的说明：
```cpp
1: SparseMatrix<double> mat(rows,cols);         // default is column major
2: mat.reserve(VectorXi::Constant(cols,6));
3: for each i,j such that v_ij != 0
4:   mat.insert(i,j) = v_ij;                    // alternative: mat.coeffRef(i,j) += v_ij;
5: mat.makeCompressed();                        // optional
```
  * 这里关键的代码是第二行，我们在每一行预留了6个供非零元素插入的空间。大多数情况下，每行或每列非零元素的个数可以有简单的方法提前预知。如果对于每个内部向量非零元素个数差别很大，那么你可能需要为每个内部向量通过给每个向量对象提供运算符[](int j)来返回第j个内部向量需要的空间来预留一定空间。如果内部向量非零元素的个数只能粗略估算，我们强烈建议你高估可能的个数而不是相反。如果违背这个建议，第一次插入新元素的时候回为每个内部向量预留两个元素的空间。
  * 第四行执行了顺序插入。这个例子中，理想的场景是第j列没有填满同事非零元素的内部索引都比i要小。这样，这个操作的复杂度将降至O(1)。
  * 调用insert(i,j)方法的时候，元素mat(i,j)必须没有被填充过，否则需要调用方法coeffRef(i,j)来安排插入元素。这个方法首先使用二分查找，如果在插入位置没找到元素，就调用insert(i,j)方法。这种方法比直接调用insert()方法更灵活但也更低效。
  * 第五行压缩了生育的空置内存同时将矩阵存储模式转换为压缩列存储模式。

## 支持的运算符和方法
由于存储格式特殊，稀疏矩阵无法像稠密矩阵那样灵活。在[Eigen]的稀疏模块中我们只暴露了从稠密矩阵全部API中仍能有效实现其功能的一部分。下文中sm代表稀疏矩阵，sv代表稀疏向量，dm代表稠密矩阵，dv代表稠密向量。

### 基础操作
---
稀疏表达式支持绝大多数的因数相关的一元和二元操作：
```cpp
sm1.real()   sm1.imag()   -sm1                    0.5*sm1
sm1+sm2      sm1-sm2      sm1.cwiseProduct(sm2)
```
然而，**这有一个大前提：存储顺序符合运算要求**。例如，下面这个例子中：
```cpp
sm4 = sm1 + sm2 + sm3;
```
sm1,sm2,sm3必须同样为行主序或列主序。另一面，sm4则没有相关要求。这意味着计算类似![](https://latex.codecogs.com/gif.latex?A%20&plus;%20A%5E%7BT%7D)的方程，![](https://latex.codecogs.com/gif.latex?A%5E%7BT%7D)必须通过合适存储顺序的临时矩阵中计算：
```cpp
SparseMatrix<double> A, B;
B = SparseMatrix<double>(A.transpose()) + A;
```
因数相关的二元运算符也可以在稀疏和稠密表达式间混用：
```cpp
sm2 = sm1.cwiseProduct(dm1);
dm2 = sm1 + dm1;
dm2 = dm1 - sm1;
```
从性能考虑，对稀疏和稠密矩阵进行加/减最好分成两步。例如，计算`dm2 = sm1 + dm1`最好写成：
```cpp
dm2 = dm1;
dm2 += sm1;
```
这个版本能充分利用稠密矩阵存储带来的高性能，同时只是在计算稀疏矩阵中少量非零元素时才进行低效计算。
稀疏表达式同样支持转置操作：
```cpp
sm1 = sm2.transpose();
sm1 = sm2.adjoint();
```
然而，这里不支持transposeInPlace()方法。

### 矩阵乘法
---
Eigen支持的稀疏矩阵乘法都总结在下面了：
  * 稀疏-稠密：
  ```cpp
  dv2 = sm1 * dv1;
  dm2 = dm1 * sm1.adjoint();
  dm2 = 2. * sm1 * dm1;
  ```
  * 对称稀疏-稠密：
  对称稀疏矩阵与一个稠密矩阵（或向量）相乘同样可以通过selfadjointView()方法指定对称来优化计算开销。
  ```cpp
  dm2 = sm1.selfadjointView<>() * dm1;        // if all coefficients of A are stored
  dm2 = A.selfadjointView<Upper>() * dm1;     // if only the upper part of A is stored
  dm2 = A.selfadjointView<Lower>() * dm1;     // if only the lower part of A is stored
  ```
  * 稀疏-稀疏
  对于稀疏矩阵相乘，有两种不同算法可以使用。默认算法是更保守同时显示保留可能出现的0元素：
  ```cpp
  sm3 = sm1 * sm2;
  sm3 = 4 * sm1.adjoint() * sm2;
  ```
  第二种算法省略掉了运行时的0元素或者小于给定阈值的元素。这个算法可以通过prune()方法激活和控制：
  ```cpp
  sm3 = (sm1 * sm2).pruned();                  // removes numerical zeros
  sm3 = (sm1 * sm2).pruned(ref);               // removes elements much smaller than ref
  sm3 = (sm1 * sm2).pruned(ref,epsilon);       // removes elements smaller than ref*epsilon
  ```
  * 交换律。
  最后，稀疏矩阵也可以应用交换律：
  ```cpp
  PermutationMatrix<Dynamic,Dynamic> P = ...;
  sm2 = P * sm1;
  sm2 = sm1 * P.inverse();
  sm2 = sm1.transpose() * P;
  ```
### 块操作
---
在读写方面，稀疏矩阵和稠密矩阵这部分支持的API是相同的，例如访问分块矩阵，访问行或者列。参看[分块操作](http://eigen.tuxfamily.org/dox/group__TutorialBlockOperations.html)了解更多。但是，处于性能考虑，向稀疏矩阵的子矩阵写入内容受到的限制更多，并且当前对于列（行）优先矩阵仅允许写入连续的列（行）构成的[稀疏矩阵][sm]。进一步，除了方法`block(...)`和`corner*(...)`，写入矩阵的属性需要在编译器就确定下来。[稀疏矩阵][sm]可以使用的写入方法总结在下面：
```cpp
SparseMatrix<double,ColMajor> sm1;
sm1.col(j) = ...;
sm1.leftCols(ncols) = ...;
sm1.middleCols(j,ncols) = ...;
sm1.rightCols(ncols) = ...;
SparseMatrix<double,RowMajor> sm2;
sm2.row(i) = ...;
sm2.topRows(nrows) = ...;
sm2.middleRows(i,nrows) = ...;
sm2.bottomRows(nrows) = ...;
```
另外，稀疏矩阵暴露的[SparseMatrixBase::innerVector()](http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrixBase.html#a65aaf3b50d205011e2bfa0de24756cce)和[SparseMatrixBase::innerVectors()](http://eigen.tuxfamily.org/dox/classEigen_1_1SparseMatrixBase.html#a3c51bf5a7eb18eab9a85949d03aed14a)方法，对列优先存储分别对应col/middleCols方法，对行优先存储分别对应row/middleRows方法。

### 三角和共轭视图
---
跟稠密矩阵一样，triangularView()方法可以用来指定一个矩阵的哪个三角部分，然后用这部分右乘稠密矩阵：
```cpp
dm2 = sm1.triangularView<Lower>(dm1);
dv2 = sm1.transpose().triangularView<Upper>(dv1);
```
selfadjointView()方法支持许多操作：
  * 优化的稀疏-稠密矩阵乘法
  ```cpp
  dm2 = sm1.selfadjointView<>() * dm1;        // if all coefficients of A are stored
  dm2 = A.selfadjointView<Upper>() * dm1;     // if only the upper part of A is stored
  dm2 = A.selfadjointView<Lower>() * dm1;     // if only the lower part of A is stored
  ```
  * 复制矩阵三角
  ```cpp
  sm2 = sm1.selfadjointView<Upper>();                               // makes a full selfadjoint matrix from the upper triangular part
  sm2.selfadjointView<Lower>() = sm1.selfadjointView<Upper>();      // copies the upper triangular part to the lower triangular part
  ```
  * 进行镜像交换
  ```cpp
  PermutationMatrix<Dynamic,Dynamic> P = ...;
  sm2 = A.selfadjointView<Upper>().twistedBy(P);                                // compute P S P' from the upper triangular part of A, and make it a full matrix
  sm2.selfadjointView<Lower>() = A.selfadjointView<Lower>().twistedBy(P);       // compute P S P' from the lower triangular part of A, and then only compute the lower part
  ```
更多支持的操作请查阅[快速查询指南](http://eigen.tuxfamily.org/dox/group__SparseQuickRefPage.html).线性求解器列表在[这里](http://eigen.tuxfamily.org/dox/group__TopicSparseSystems.html)
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