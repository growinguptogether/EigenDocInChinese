# 固定大小可量化的Eigen对象
---
这一节主要解释定语“固定大小可量化”的具体含义。

## 总结
当一个Eigen对象的大小是固定的，并且大小是16字节的整数倍，那么我们就称它为“固定大小可量化”。
以下是一些例子：
- [Eigen::Vector2d]
- [Eigen::Vector4d]
- [Eigen::Vector4f]
- [Eigen::Matrix2d]
- [Eigen::Matrix2f]
- [Eigen::Matrix4d]
- [Eigen::Matrix4f]
- [Eigen::Affine3d]
- [Eigen::Affine3f]
- [Eigen::Quatentiond]
- [Eigen::Quatentionf]

## 解释

首先，我们需要明确“固定大小”的概念：如果一个Eigen对象的行数和列数在编译期就固定下来了，那么我们就称其为“固定大小”。所以，我们称Matrix3f是固定大小的，而MatrixXf不是（与固定大小相对的是动态大小）。
固定大小的Eigen对象参数构成的数组其实是一个简单的“静态数组”，这个数组不会在运行期间动态生成。例如一个Matrix4f对象背后的实现其实就是一个`float array[16]`。
一般来说，固定大小的对象都非常小，所以，我们希望以无额外开销运行的方式处理这些对象。额外开销的概念包含内存开销和代码执行的时间开销。
现在，代码量化（包括SSE和AltiVec）目标都是将可执行代码打包成128位的代码包。随之而来，为了提高代码运行效率，这些代码包需要按128位对齐。
因此，[Eigen]对象能被量化的必要条件就是，它的大小需要是128位或者说16个字节的整数倍。所以[Eigen]要求它的对象必须16字节对齐，随之而来的是程序会认为这些对象已经进行了对齐，并且不会在运行时再次检查对齐操作是否有执行过。

[Eigen]:http://eigen.tuxfamily.org/dox/namespaceEigen.html