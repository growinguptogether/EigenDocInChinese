# 开始

> ref: http://eigen.tuxfamily.org/dox/GettingStarted.html

本节是eigen的入门教程。它有两个目的。1. 对于想尽快借助eigen进行编码的人，提供了一份尽可能精简的介绍。2. 你也可以将下面的内容作为完整教程的导引，在完整教程中，我们将对Eigen库进行更加详细的介绍；那么当你读完这一部分的内容后，请继续阅读[矩阵类](./TheMatrixClass.md)的内容。

## 如何“安装”Eigen

你只需要下载和解压Eigen的源代码，就可以直接使用它，在[wiki](http://eigen.tuxfamily.org/index.php?title=Main_Page#Download)里面有下载的指导。事实上，子目录Eigen中的头文件，就是程序中使用Eigen所要依赖的全部内容。对于不同平台，依赖的头文件都是一样的。不需要使用CMake或安装其他的任何别的东西。

## 第一个简单的程序

下面有一个非常简单的程序作为开始。

```c++
#include <iostream>
#include <Eigen/Dense>

using Eigen::MatrixXd;

int main()
{
    MatrixXd m(2,2);
    m(0,0) = 3;
    m(1,0) = 2.5;
    m(0,1) = -1;
    m(1,1) = m(1,0) + m(0,1);
    std::cout << m << std::endl;
}
```

## 编译和运行你的第一个程序

这里没有库需要连接。在编译上述程序的时候，你需要记住的唯一的事情就是让编译器能够找到Eigen的头文件在哪。你放置Eigen源代码的目录必须在include路径下。使用gcc，你可以使用-I选项设定include路径，因此你可以用下面的命令进行编译：

    g++ -I /path/to/eigen/ my_program.cpp -o my_program

> 在window平台，可以使用如下命令
> 
> `cl /Ipath/to/eigen my_program.cpp /Femy_program.exe`

当运行程序的时候，产生如下的结果：

```
  3  -1
2.5 1.5
```

## 第一个程序的解释

Eigen头文件中定义了很多类型，但是对于简单的程序一般只要使用`MatrixXd`类型就可以了。这个类型可以用来表示任 大小的矩阵（MatrixXd中的X），其中每个元素的类型是double类型（MatrixXd中的d）。参考[快速指导](./QuickReferenceGuide.md)， 里面包含了你能用来描述矩阵对象的各种类型的概述。

`Eigen/Dense`头文件中定义了MatrixXd类型的所有成员函数和相关类型。(详细可以在[头文件列表](Add Link)中查看)。所有Eigen头文件中定义的类和函数都在Eigen命名空间中。

main函数中的第一行定义了MatrixXd为类型的变量，是一个2行2列的矩阵，内部的元素没有进行初始化。表达式`m(0,0)=3`，将左上方的元素设定为3.你需要使用圆括号来引用矩阵中的元素。按计算机科学的惯例，第一个index是0，这与数学惯例，第一个index是1不同。

接下来的三行是设定其他不同的元素。最后一行是将矩阵打印到标准输出流。

## Example 2： 矩阵和向量

下面是另一个例子，包含了向量和矩阵的使用。

```c++
// 大小在运行时确定
#include <iostream>
#include <Eigen/Dense>

using namespace Eigen;
using namespace std;

int main()
{
  MatrixXd m = MatrixXd::Random(3,3);           // Matrix3d m = Matrix3d::Random();
  m = (m + MatrixXd::Constant(3,3,1.2)) * 50;   // m = (m + Matrix3d::Constant(1.2)) * 50;
  cout << "m =" << endl << m << endl;           // 
  VectorXd v(3);                                // Vector3d v(1,2,3);
  v << 1, 2, 3;
  cout << "m * v =" << endl << m * v << endl;
}
```

输出结果是：

```
m =
  94 89.8 43.5
49.4  101 86.8
88.3 29.8 37.8
m * v =
404
512
261
```

## 第二个程序的解释

第二个程序一开始声明了一个3乘3的矩阵m，然后使用Random()生成-1到1的随机数进行初始化。第二行进行线性映射，将值变换为10到110之间。函数`MatrixXd::Constant(3,3,1.2)`得到一个3乘3，每个元素均为1.2的矩阵。剩下的是常规的计算。

下一行，引入了一个新的类型`VectorXd`，表示任意大小的列向量。这里声明了拥有三个元素的列向量，但是内部的元素没有初始化。接下来的一行是逗号初始化，在[高级初始化](./AdvancedInitialization.md)中继续讲解。初始化完之后，得到的向量v如下：

```math
v = \begin{bmatrix}
1\\
2\\ 
3
\end{bmatrix}
```

![](http://latex.codecogs.com/gif.latex?v%20%3D%20%5Cbegin%7Bbmatrix%7D%201%5C%5C%202%5C%5C%203%20%5Cend%7Bbmatrix%7D)

最后一行是将矩阵和向量进行相乘，并且输出结果。

在回过来看第二个示例程序。使用了MatrixXd来表示任意大小的矩阵。对应的注释的代码中，使用了Matrix3d，表示固定大小3x3的矩阵。由于类型名称已经给定了矩阵的大小，在构造函数中不用再次说明矩阵大小。类似的还有Vector3d。注意，Vector3d对应的每个元素，在构造函数中进行了设定。

使用固定大小的矩阵和向量有两个好处。因为编译器在编译期就能知道矩阵和向量的大小，编译器能够生成出更好更快的代码。提前指定大小可以允许编译器在编译期执行更为严格的逻辑检查。比如当矩阵Matrix4d和向量Vector3d相乘的时候，编译器就会报错。但是，使用这些固定大小的类型会增加编译时间和生成的可执行文件的大小。还有些矩阵的大小可能在编译期间无法确定。一个推荐的方案是如果矩阵大小不大于4x4的时候使用固定大小类型。

## 接下来该干嘛？

花点时间继续读完[全部教程](./TheMatrixClass.md)是非常有必要的。

但是如果你觉得没有必要，那么你可以直接查阅类文档和我们的[快速参考指导](./QuickReferenceGuide.md)。

- Next：[矩阵类](./TheMatrixClass.md)
