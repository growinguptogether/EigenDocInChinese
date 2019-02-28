
当前版本的Eigen没有提供方便直接调用的切片或修改矩阵维度的方法。但是，（如果要使用）这些特性，使用[Map]( TODO add link)类可以非常容易自己实现。

## Reshape

修改（矩阵）维度操作包含两个含义：修改矩阵的大小（维度）；保持原矩阵的系数。因为作为入参的矩阵的维度无法在编译期就确定下来，与其修改输入的矩阵本身，
不如在方法调用的时候使用[MAP]( TODO add link)类创建一个新的实例更为合适。以下是创建一个一维线性矩阵的例子

```c++
MatrixXf M1(3,3);  //列优先存储
M1 << 1, 2, 3,
      4, 5, 6,
      7, 8, 9;
Map<RowVecorXf> v1(M1.data(),M1.size());
cout << "v1:" << endl << v1 << endl;
Matrix<float,Dynamic,Dynamic,RowMajor> M2(M1);
Map<RowVecotrXf> v2(M2.data(), M2.size());
cout << "v2:" << endl << v2 << endl;

[output:]

v1:
1 4 7 2 5 8 3 6 9
v2:
1 2 3 4 5 6 7 8 9
```
注意输入矩阵系数排列顺序改变的情况下，转化为线性存储后各元素存储位置的变化规律。以下是另一个将2X6矩阵转换为6X2矩阵的例子
```c++
MatrixXf M1(2,6);  //列优先存储
M1 << 1, 2, 3,  4,  5,  6,
      7, 8, 9, 10, 11, 12;
Map<MatrixXf> M2(M1.data(),6,2);
cout << "M2" << endl << M2 << endl;

[output:]
M2:
1 4
7 10
2 5
8 11
3 6
9 12
```
## Slicing

切片操作的定义是从矩阵中抽取一些行/列/元素。同样，类[Map]( TODO add link)可以帮助我们轻松实现这个特性。

例如，从一个向量中每间隔P个元素剔除一个元素：
```c++
RowVectorXf v = RowVectorXf::LinSpaced(20,0,19);
cout << "Input:" << endl << v << endl;
Map<RowVectorXf,0,InnerStride<2> > v2(v.data(), v.size()/2);
cout << "Even:" << v2 << endl;

output:
Input:
0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19
Even: 0  2  4  6  8 10 12 14 16 18
```
你也可以根据你的矩阵存储顺序对应使用outer-stride或者inner-stride来实现从矩阵中每隔3列取出一列。

```c++
MatrixXf M1 = MatrixXf::Random(3,8);
cout << "Column major input:" << endl << M1 << "\n";

typedef Matrix<float,Dynamic,Dynamic,RowMajor> RowMajorMatrixXf;
RowMajorMatrixXf M3(M1);
cout << "Row Major input:" << endl << M3 << "n\";
Map<RowMajorMatrixXf,0,Stride<Dynamic,3> > M4(M3.data(), M3.rows(), (M3.cols()+2/3, Stride<Dynamic,3>(M3.outerStride(),3));
cout << "1 column over 3:" << endl << M4 << "\n";

[output:]
Column major input:
  0.68   0.597  -0.33   0.108  -0.27   0.832  -0.717  -0.514
 -0.211  0.823   0.536 -0.0452  0.0268 0.271   0.214  -0.726
  0.566 -0.605  -0.444  0.258   0.904  0.435  -0.967   0.608
  1 column over 3:
  0.68   0.108  -0.717
 -0.211 -0.0452   0.214
  0.566   0.258  -0.967
Row major input:
  0.68   0.597  -0.33   0.108  -0.27   0.832  -0.717  -0.514
 -0.211  0.823   0.536 -0.0452  0.0268 0.271   0.214  -0.726
  0.566 -0.605  -0.444  0.258   0.904  0.435  -0.967   0.608
1 column over 3:
  0.68   0.108  -0.717
 -0.211 -0.0452  0.214
  0.566  0.258  -0.967
```
- Previous: [Interfacing with raw buffers: the Map class](./TheMapClass)
- Next：[混淆(Aliasing)](./Aliasing.md)
