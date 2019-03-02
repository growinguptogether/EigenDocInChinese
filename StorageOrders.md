# 存储顺序（Storage orders）
- [CheapterMenu](TODO add link)
---
矩阵和二维数组有两种不同的存储顺序：列优先和行优先顺序。本节将解释这两种存储顺序的含义，然后介绍如何在程序中指定采用哪种顺序的方法。
## 列优先和行优先(Column-major and row-major storage)
矩阵从形式上看是一个二维网格。然而，矩阵对象在内存中的存储形式只能是一维线性存储。因此，我们就有了两种方式存储矩阵，按行存储或者按列存储。
如果当一个矩阵是一行一行的顺序存储的时候，我们称这个矩阵是[行优先]存储。此时，只有当前行的元素都存储完成后才开始保存下一行，以此类推。例如下面的矩阵

![](http://latex.codecogs.com/gif.latex?%5Cinline%20%5Cbegin%7Bbmatrix%7D%20%268%20%262%20%262%20%269%20%5C%5C%20%269%20%261%20%264%20%264%20%5C%5C%20%263%20%265%20%264%20%265%20%5Cend%7Bbmatrix%7D)

如果这个矩阵是行优先存储，那么内存里元素分布的顺序如下：
`8 2 2 9 9 1 4 4 3 5 4 5`
相对的，如果一个矩阵的存储顺序是一列列的，即第一列保存完成后再保存第二列，如此递归。如果上述矩阵是使用列优先存储，那么其内存分布如下：
`8 9 3 2 1 5 2 4 4 9 4 5`
代码示例在[Eigen]（TODO add link)里查看。这里使用了[PlainObjectBase::data()](TODO add link)函数，该函数返回值是指向矩阵首元素在内存存储的地址。
```c++
Matrix<int, 3, 4, ColMajor> Acolmajor;
Acolmajor << 8, 2, 2, 9,
             9, 1, 4, 4,
             3, 5, 4, 5;
cout << "The matrix A:" << endl;
cout << Acolmajor << endl << endl; 
cout << "In memory (column-major):" << endl;
for (int i = 0; i < Acolmajor.size(); i++)
  cout << *(Acolmajor.data() + i) << "  ";
cout << endl << endl;
Matrix<int, 3, 4, RowMajor> Arowmajor = Acolmajor;
cout << "In memory (row-major):" << endl;
for (int i = 0; i < Arowmajor.size(); i++)
  cout << *(Arowmajor.data() + i) << "  ";
cout << endl;

//output
The matrix A:
8 2 2 9
9 1 4 4
3 5 4 5

In memory (column-major):
8  9  3  2  1  5  2  4  4  9  4  5  

In memory (row-major):
8  2  2  9  9  1  4  4  3  5  4  5  
```
## Eigen里的存储顺序(Storage orders in Eigen)

矩阵或二维数组的存储顺序可以通过模板[Matrix](TODO add link)或者[Array](TODO add link)的参数*Options*来进行指定。正如[Matrix](TODO add link)类解释的，Matrix类模板有6个模板参数，其中3个为必填参数(*Scalar*,*RowsAtCompileTime*,*ColsAtCompileTime*),还有3个可选参数(*Options*,*MaxRowsAtCompileTime*,*MaxColsAtCompileTime*)。如果*Options*参数被设置成*RowMajor*，此时矩阵或数组就是按照行优先顺序存储；如果被设置成*ColMajor*，此时就按照列优先存储。这项机制在[Eigen](TODO add link)程序中被用来指定存储顺序。

如果存储顺序未明确指定，[Eigen](TODO add link)中默认的参数是列优先。在eigen定义的类型别名(例如Matrix3f,ArrayXXd,等等)中也是使用的这个默认值。

正不同存储顺序的矩阵和数组之间可以相互赋值，正如上面的例子展示的，矩阵*Arowmajor*可以被*Acolmajor*行初始化。[Eigen](TODO add link)将自动调整对象的存储顺序。甚至如果我们希望，我们可以在表达式中将行优先和列优先矩阵混用。
## 我应该采用哪种存储顺序呢?

所以，在你的程序里应该如何选择存储顺序呢？这不是一个简单可以回答的问题——这与你的使用场景相关。一下是一些你需要关注的关键点：

- 或许你的客户会要求你使用他们指定的存储顺序。或者你同时使用的[Eigen](TODO add link)之外的三方库，这些三方库指定了自己的存储顺序。这些场景下，你要使用哪个存储顺序，答案是最简单的。
- 当矩阵是行优先存储的时候，一行一行遍历矩阵的算法执行效率更高，因为元素存储顺序与算法更贴合。同样，当算法是一列一列遍历矩阵的时候，采用列优先效果会更好。所以，在做决定之前进行性能测试，比较那种方式更适合你的程序，这或许是一个不错的做法。
- [Eigen](TODO add link)的默认存储顺序是列优先。自然，[Eigen](TODO add link)库的开发和测试工作都是基于列优先矩阵进行的。这意味着，虽然我们希望存储顺序对用户透明，但是[Eigen](TODO add link)针对列优先矩阵的处理性能或许会更优秀。
- previous: [Aliasing](./Aliasing.md)
- next: [Alignment Issue](TODO add link)
