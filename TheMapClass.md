# 与raw缓冲区对接：Map类

这一页介绍了怎么和"raw" c/c++数组一块干活。这个在很多方面是有用的，
特别是需要从其他的库中将向量或矩阵“导入”到Eigen的时候。

## 介绍

有时候，你想要将你事先定义好的数的数组作为Eigen里面的向量或矩阵使用。然而，
一个方案是拷贝一份数据，但是更多的时候，你想要复用数据内存，直接作为Eigen的类型进行使用。
幸运的是，使用Map类可以很轻松的实现。

## Map类型和声明Map变量

一个Map对象在Eigen中的定义如下：

```c++
Map<Matrix<typename Scalar, int RowsAtCompileTime, int ColsAtCompileTime>>
```

注意的是，在默认情况下，Map只需要单个模板参数就可以了。

为了构建Map对象，还需要两个准备：一个值像定义好的元素数组内存区域的指针，
和期望的矩阵或向量的大小。比如，定义一个float类型，运行期间确定大小的矩阵，可以：

```c++
Map<MatrixXf> mf(pf, rows, columns);
```

这里，pf是`float *`指向数组内存。一个固定大小只读的向量可以声明如下：

```c++
Map<const Vector4i> mi(pi);
```

这里pi是`int *`类型。这种情况下，大小就不需要传递给构造函数，因为类型中已经给出了大小。

需要注意的是，Map没有默认构造函数。你必须传递一个指针来初始化对象。但是，
你可以绕开这个需求。（see 改变mapped数组）

Map已经足够灵活，能够包容各种各样的数据。这里有另外两个可选的模板参数。

```c++
Map <typename MatrixType, int MapOptions, typename StrideType>
```

- MapOptions 用来表示指针是对齐的还是非对齐的，默认是非对齐的。 TODO add link
- StrideType 允许你使用Stride类对数据内存的布局进行特殊自定义。一个例子
将数据数组以行优先的格式进行布局：

```c++
int array[8];
for (int i=0; i<8; ++i) array[i] = i;
cout << "Column-major: \n" << Map<Matrix<int,2,4>>(array) << endl;
cout << "Row-major:\n" << Map<Matrix<int,2,4,RowMajor>>(array) << endl;
cout << "Row-major using stride:\n" << Map<Matrix<int,2,4>, Unaligned, Stride<1,4>>(array) << endl;

// output
Column-major:
0 2 4 6
1 3 5 7
Row-major:
0 1 2 3
4 5 6 7
Row-major using stride:
0 1 2 3
4 5 6 7
```

但是，Stride比这里给出的更加灵活，更多内容见类说明。

## 使用Map变量

你可以像其他Eigen类型一样使用Map对象。

```c++
typedef Matrix<float,1,Dynamic> MatrixType;
typedef Map<MatrixType> MapType;
typedef Map<cosnt MatrixType> MapTypeConst; // a read only map 
const int n_dims = 5;

MatrixType m1(n_dims), m2(n_dims);
m1.setRandom();
m2.setRandom();
float *a = &m2(0); // get address storing the data for m2 
MapType m2map(p,m2.size()); // m2map shares data with m2 
MapTypeConst m2mapconst(p,m2.size()); // a readonly access for m2 

cout << "m1: " << m1 << endl;
cout << "m2: " << m2 << endl;
cout << "Squared euclidean distance: " << (m1 - m2).squaredNorm() << endl;

cout << "Squared euclidean distance, using map: " << (m1-m2map).squaredNorm() << endl;
m2map(3) = 7; // this will change m2, since they share the same array
cout << "Update m2: " << m2 << endl;
cout << "m2 coefficient 2, constant accessor: " << m2mapconst(2) << endl;
/* m2mapconst(2) = 5; */ // this yields compile-time error

// output
m1:   0.68 -0.211  0.566  0.597  0.823
m2: -0.605  -0.33  0.536 -0.444  0.108
Squared euclidean distance: 3.26
Squared euclidean distance, using map: 3.26
Updated m2: -0.605  -0.33  0.536      7  0.108
m2 coefficient 2, constant accessor: 0.536
```

所有的Eigen函数去访问Map对象的时候，和使用Eigen的其它类型一样。但是，
当书写Eigen类型作为参数的函数的时候，类型转换并不会自动发生：Map类型并不与Dense类型完全相同。
详细见：`Writing Functions Taking Eigen Types as Parameters` TODO add link

## 改变Mapped数组

在声明Map对象之后，也能够改变Map，使用c++的`placement new`格式，用来在已经分配的内存中，
创建对象。

```c++
int data[] = {1,2,3,4,5,6,7,8,9};
Map<RowVectorXi> v(data,4);
cout << "The mapped vector v is: " << v << "\n";
new (&v) Map<RowVectorXi>(data+4,5);
cout << "Now v is: " << v << "\n";
```

这个过程并没有调用内存分配，语法的格式从现有的内存中构建对象。

这种实现方式，可以在实现不知道mapped数组在内存中具体位置的时候声明一个Map对象。

```c++
Map<Matrix3f> A(NULL); // don't try to use this matrix yet!
VectorXf b(n_matrices);
for (int i = 0; i < n_matrices; i++)
{
    new (&A) Map<Matrix3f>(get_matrix_pointer(i));
    b(i) = A.trace();
}
```
- previous: []()
- next: [Reshape and Slicing](./ReshapeAndSlicing)
