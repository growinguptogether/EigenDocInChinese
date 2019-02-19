# 高阶初始化

这一页讨论了几个初始化矩阵的高阶方法。更加详细的介绍了之前提到过的逗号初始化。
同时，也说明了如何初始化特殊矩阵，如单位矩阵和零矩阵。

## 逗号初始化

为了更方便的对矩阵或数组或向量的所有元素进行初始化，Eigen提供了逗号初始化的方式。
只需要简单的将元素列出来，从左上角开始，从左到右，从上到下，依次列出。
对象的大小需要提前设定。如果提供的元素个数偏少，或偏多，Eigen都会抱怨。

```c++
Matrix3f m;
m << 1,2,3,
     4,5,6,
     7,8,9;
std::cout << m;

// output 
1 2 3 
4 5 6
7 8 9
```

此外，初始化列表中的元素可以是向量或是矩阵。这种常见的形式是将向量或矩阵连接起来。
比如，下面例子给出了如何将两个向量连接到一块。需要注意的是，使用逗号初始化之前，
必须设定大小。

```c++
RowVectorXd vec1(3);
vec1 << 1,2,3;
std::cout << "vec1 = " << vec1 << std::endl;

RowVectorXd vec2(4);
vec2 << 1,4,6,9;
std::cout << "vec2 = " << vec2 << std::endl;

RowVectorXd joined(7);
joined << vec1, vec2;
std::cout << "joined = " << joined << std::endl;

// output 
vec1 = 1 2 3
vec2 =  1  4  9 16
joined =  1  2  3  1  4  9 16
```

我们可以使用相同的技术来初始化矩阵。

```c++
MatrixXf matA(2,2);
matA << 1,2,3,4;
MatrixXf matB(4,4);
matB << matA, matA/10, matA/10, matA;
std::cout << matB << std::endl;

// output 
1   2 0.1 0.2
  3   4 0.3 0.4
0.1 0.2   1   2
0.3 0.4   3   4
```

逗号初始化的方式也可用来对块中的数据进行填充，如`m.row(i)`。
下面给出了相对于第一个例子更加复杂的实现方式：

```c++
Matrix3f m;
m.row(0) << 1,2,3;
m.block(1,0,2,2) << 4,5,7,8;
m.col(2).tail(2) << 6,9;
std::cout << m;

// output
1 2 3 
4 5 6
7 8 9
```

## 特殊的矩阵和向量

矩阵或数组有像`Zero()`的静态方法，用来将所有的元素初始化成零。这里有三种形态。
第一种不需要传递参数，仅适用于固定大小的对象。如果你需要对动态大小的对象初始化成0，
你需要设定大小。因此，第二种形态需要传入一个参数，适用于一维动态大小的对象。
第三种形态是需要传入两个参数，适用于二维对象。三种使用方式见下面的例子：

```c++
cout << "A fixed-size array:\n";
Array33f a1 = Array33f::Zero();
cout << a1 << "\n\n";

cout << "A one-dimensional dynamic-size array:\n";
ArrayXf a2 = ArrayXf::Zero(3);
cout << a2 << "\n\n";

cout << "A tow-dimensional dynamic-size array:\n";
ArrayXXf a3 = ArrayXXf::Zero(3,4);
cout << a3 << "\n";

// output 
A fixed-size array:
0 0 0
0 0 0
0 0 0

A one-dimensional dynamic-size array:
0
0
0

A two-dimensional dynamic-size array:
0 0 0 0
0 0 0 0
0 0 0 0
```
