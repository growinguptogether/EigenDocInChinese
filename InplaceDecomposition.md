# 矩阵原地分解
---
从Eigen3.3开始，LU、Cholesky和QE分解都可以在原地实现，也就是可以直接在输入矩阵的内部完成。当输入矩阵非常巨大或者机器内存资源受限的场景（嵌入式系统）下，这种特性非常有用。

为此，对应需要的分解类必须用`Ref<>`矩阵类型实例化，同时分解类对象必须将待分解矩阵作为入参进行构造。像下面的例子，我们打算对输入矩阵原地进行部分旋转LU分解。

首先，我们准备好原材料，定义一个2X2的矩阵A:
```cpp
#include <iostream>
#include <Eigen/Dense>
using namespace std;
using namespace Eigen;
int main()
{
  MatrixXd A(2,2);
  A << 2, -1, 1, 3;
  cout << "Here is the input matrix A before decomposition:\n" << A << endl;
```
输出：
```sh
Here is the input matrix A before decomposition:
 2 -1
 1  3
```
没有任何问题。然后，让我们定义原地LU分解对象`lu`，接着检查矩阵A的内容：
```cpp
  PartialPivLU<Ref<MatrixXd> > lu(A);
  cout << "Here is the input matrix A after decomposition:\n" << A << endl;
```
输出：
```sh
Here is the input matrix A after decomposition:
  2  -1
0.5 3.5
```
此时，lu对象在矩阵A的内存空间里计算并保存了L和U两个分量。矩阵A的参数在因式分解的过程中被破坏了，然后被L和U的参数所替代。你可以验证：
```cpp
cout << "Here is the matrix storing the L and U factors:\n" << lu.matrixLU() << endl;
```
```sh
Here is the matrix storing the L and U factors:
  2  -1
0.5 3.5
```
现在，你可以正常使用lu对象，例如解方程`Ax = b`：
```cpp
  MatrixXd A0(2,2); A0 << 2, -1, 1, 3;
  VectorXd b(2);    b << 1, 2;
  VectorXd x = lu.solve(b);
  cout << "Residual: " << (A0 * x - b).norm() << endl;

output:
Residual: 0
```
这里，因为原矩阵A的内容被破坏了，我们需要定义另一个矩阵A0来验证结果。
因为A和lu的内存空间是共享的，修改矩阵A会导致lu失效。我们修改A的内容然后尝试解刚才那个方程，很容易可以验证我们的论述：
```cpp
  A << 3, 4, -2, 1;
  x = lu.solve(b);
  cout << "Residual: " << (A0 * x - b).norm() << endl;

output:
Residual: 15.8114
```
请注意，这里我们没有使用智能指针，**用户有义务**保证入参A的内存空间在对象lu的生存期间内都是合法的。
如果你想对修改过的A重新进行因式分解，你需要正常调用compute方法就可以了：
```cpp
  A0 = A; // save A
  lu.compute(A);
  x = lu.solve(b);
  cout << "Residual: " << (A0 * x - b).norm() << endl;

output:
Residual: 0
```
请注意，调用compute方法并不会修改lu对象指向的内存地址。如果，compute方法的入参不是A而是A1，A1的内容并不会被修改。存放A1矩阵LU分解结果的内存空间仍将是矩阵A的空间。这可以很简单的进行证实：
```cpp
  MatrixXd A1(2,2);
  A1 << 5,-2,3,4;
  lu.compute(A1);
  cout << "Here is the input matrix A1 after decomposition:\n" << A1 << endl;

output:	
Here is the input matrix A1 after decomposition:
 5 -2
 3  4
```

矩阵A1没有变化，你也可以解方程`A1*x = b`，然后直接用A1来验证余数：
```cpp
  x = lu.solve(b);
  cout << "Residual: " << (A1 * x - b).norm() << endl;

output:
Residual: 2.48253e-16
```

下面是支持原地分解机制的分解类：
- [LLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LLT.html)
- [LDLT](http://eigen.tuxfamily.org/dox/classEigen_1_1LDLT.html)
- [PartialPivLU](http://eigen.tuxfamily.org/dox/classEigen_1_1PartialPivLU.html)
- [FullPivLU](http://eigen.tuxfamily.org/dox/classEigen_1_1FullPivLU.html)
- [HouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1HouseholderQR.html)
- [ColPivHouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1ColPivHouseholderQR.html)
- [FullPivHouseholderQR](http://eigen.tuxfamily.org/dox/classEigen_1_1FullPivHouseholderQR.html)
- [CompleteOrthogonalDecomposition](http://eigen.tuxfamily.org/dox/classEigen_1_1CompleteOrthogonalDecomposition.html)