# 混淆 Aliasing

在Eigen，混淆指的是相同的矩阵（或数组或向量）出现在赋值操作符的左边和右边。如下表达式，`mat = 2*mat`或者`mat = mat.transpose()`。第一个表达式是没有问题的，但是第二个表达式，会出现不可预料的结果。这一页会解释什么是混淆，以及他的危害是什么，怎么进行处理。

## 例子

下面是简单的显示混淆的例子：

```c++
MatrixXi mat(3,3);
mat << 1,2,3  4,5,6, 7,8,9;
cout << "Here is the matrix mat:\n" << mat << endl;

// This assignment shows the aliasing problem
mat.bottomRightCorner(2,2) = mat.topLeftCorner(2,2);
cout << "After the assignment, mat = \n" << mat << endl;

// output
Here is the matrix mat:
1 2 3
4 5 6
7 8 9
After the assignment, mat = 
1 2 3
4 1 2
7 4 1
```

可以发现赋值后的结果和预期是不一样的。问题出现在这个表达式：

```c++
mat.bottomRightCorner(2,2) = mat.topLeftCorner(2,2);
```

这个表达式表现出来的混淆是：`mat(1,1)`既出现在`mat.bottomRightCorner(2,2)`区块中，
又出现在`mat.topLeftCorner(2,2)`区块中。在赋值之后，右下角(2,2)出现的值应该是，
赋值前mat(1,1)的值，也就是5。但是结果输出的却是1。这个问题的出现是因为，
Eigen使用了惰性求解。这个结果类似于：

```c++
mat(1,1) = mat(0,0);
mat(1,2) = mat(0,1);
mat(2,1) = mat(1,0);
mat(2,2) = mat(1,1);
```

因此，赋值给mat(2,2)的是已经被修改的mat(1,1)的值。下一节会介绍如何使用`eval()`解决这个问题。

当试图缩小矩阵的时候，混淆会出现的更加频繁。比如：`vec = vec.head(n)`和`mat = mat.block(i,j,r,c)`。

通常情况下，在编译期间无法检测出混淆的问题：如果在第一个例子中，mat如果大一点，
那么就不会出现上述重叠的问题，那么将不会有混淆的问题。但是，Eigen在运行时，会检测出一些混淆问题。
下面的显示混淆在，[矩阵和向量运算](./MatrixandVectorArithmetic.md)中出现过：

```c++
Matrix2i a; a << 1,2,3,4;
cout << "Here is the matrix a:\n" << a << endl;

a = a.transpose(); // !!! do NOT do this !!!
cout << "and the result of the aliasing effect:\n" << a << endl;

// output
Here is the matrix a:
1 2
3 4
and the result of the aliasing effect:
1 2
2 4
```

结果输出了混淆问题。但是，Eigen在默认情况下会进行运行时检测，并提供类似下面的消息：

```
void Eigen::DenseBase<Derived>::checkTransposeAliasing(const OtherDerived&) const 
[with OtherDerived = Eigen::Transpose<Eigen::Matrix<int, 2, 2, 0, 2, 2> >, Derived = Eigen::Matrix<int, 2, 2, 0, 2, 2>]: 
Assertion `(!internal::check_transpose_aliasing_selector<Scalar,internal::blas_traits<Derived>::IsTransposed,OtherDerived>::run(internal::extract_data(derived()), other)) 
&& "aliasing detected during transposition, use transposeInPlace() or evaluate the rhs into a temporary using .eval()"' failed.
```

用户可以使用`EIGEN_NO_DEBUG`宏，取消在运行时对混淆问题的检测。上述的例子中，为了更好的说明问题，
使用该宏将混淆问题检测关闭了。关于Eigen运行时检测，详细见 Assertions, TODO add link

## 解决混淆问题

如果你理解引起混淆问题的原因，那么就应该知道如何解决这个问题：Eigen需要将等号右边
的值预先计算并存储到临时变量，然后将这个变量赋值给等号左边。`eval()`表达式就是做这事情的。

举个例子，下面是第一个例子的正确形式：

```c++
MatrixXi mat(3,3);
mat << 1,2,3,  4,5,6,  7,8,9;
cout << "Here is the matrix mat:\n" << mat << endl;

// The eval() solve the aliasing problem
mat.bottomRightCorner(2,2) = mat.topLeftCorner(2,2).eval();
cout << "After the assignment, mat = \n" << mat << endl;

// output
Here is the matrix mat:
1 2 3
4 5 6
7 8 9
After the assignment, mat = 
1 2 3
4 1 2
7 4 5
```

现在mat(2,2)得到了正确的值，5。

对于第二个例子中，也可以用同样的方式进行处理，使用`a = a.transpose().eval()`替换掉`a = a.transpose()`即可。
但是，通常情况下，更加好的方式是使用Eigen提供的另一个函数`transposeInPlace()`。显示如下：

```c++
MatrixXf a(2,3); a << 1,2,3,4,5,6;
cout << "Here is the initial matrix a:\n" << a << endl;

a.transposeInPlace();
cout << "and after being transposed:\n" << a << endl;

// output
Here is the initial matrix a:
1 2 3
4 5 6
and after being transposed:
1 4
2 5
3 6
```

如果类似`xxxInPlace()`函数存在，那么优先使用它，因为这样的表达方式更加的清晰。同时也允许Eigen内部进行更多的优化。`xxxInPlace()`类似的函数有：

| Original function | In-place function |
|-------------------|-------------------|
| MatrixBase::adjoint() | MatrixBase::adjointInPlace() |
| DenseBase::reverse() | DenseBase::reverseInPlace() |
| LDLT::solve() | LDLT::solveInPlace() |
| LLT::solve() | LLT::solveInPlace() |
| TriangularView::solve() | TriangularView::solveInPlace() |
| DenseBase::transpose() | DenseBase::transposeInPlace() |

对于如`vec = vec.head(n)`，向量或矩阵收缩的例子，可以使用`conservativeResize()`.

## 混淆和component-wise操作

正如上面的例子所述，对于相同的矩阵或数组出现在赋值操作符的左边和右边，可能会引起意外，
通常需要将等号右边的通过`eval()`进行显示执行来进行避免。但是，component-wise操作是安全的，
如（矩阵加，标量乘，数组乘法等）。

下面的例子仅仅是component-wise操作，因此不需要eval。

```c++
MatrixXf mat(2,2); 
mat << 1, 2,  4, 7;
cout << "Here is the matrix mat:\n" << mat << endl << endl;
mat = 2 * mat;
cout << "After 'mat = 2 * mat', mat = \n" << mat << endl << endl;
mat = mat - MatrixXf::Identity(2,2);
cout << "After the subtraction, it becomes\n" << mat << endl << endl;
ArrayXXf arr = mat;
arr = arr.square();
cout << "After squaring, it becomes\n" << arr << endl << endl;
// Combining all operations in one statement:
mat << 1, 2,  4, 7;
mat = (2 * mat - MatrixXf::Identity(2,2)).array().square();
cout << "Doing everything at once yields\n" << mat << endl << endl;

// output
Here is the matrix mat:
1 2
4 7

After 'mat = 2 * mat', mat = 
 2  4
 8 14

After the subtraction, it becomes
 1  4
 8 13

After squaring, it becomes
  1  16
 64 169

Doing everything at once yields
  1  16
 64 169
```

通常情况下，如果右边表达式(i,j)位置的值仅仅依赖于左边表达式对应(i,j)位置的值，那么这个赋值操作是安全的。
这种情况下，就不需要对等号右边的表达式就不需要提前显示执行。
