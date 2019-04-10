# Introduction

本项目是针对**Eigen3.3.7**进行的翻译项目，其中不包括对Reference和Class List的翻译。Eigen3.3.7对应的官方文档的地址为：http://eigen.tuxfamily.org/dox/ 。如果该网址不能访问，可以直接下载[Eigen的英文文档](./eigen-doc-eng.tgz)

## Content

- [x] 1. [Getting started - 入门开始](./GettingStarted.md)
- [ ] 2. Dense matrix and array manipulation - 稠密矩阵和数组处理
  - [x] 2.1 [The Matrix class - 矩阵类](./TheMatrixClass.md)
  - [x] 2.2 [Matrix and vector arithmetic - 矩阵和向量运算](./MatrixandVectorArithmetic.md) 
  - [x] 2.3 [The Array class and coefficient-wise operations - 数组类和coefficient-wise操作](TheArrayClassAndCoefficientWiseOperations.md)
  - [x] 2.4 [Block operations - 块运算](./BlockOperations.md)
  - [x] 2.5 [Advanced initialization - 高阶初始化](./AdvancedInitialization.md)
  - [x] 2.6 [Reductions, visitors and broadcasting](./ReductionsVisitorsAndBroadcasting.md)
  - [x] 2.7 [Interfacing with raw buffers: the Map class - 与raw缓冲区对接：Map类](./TheMapClass.md)
  - [x] 2.8 [Reshape and Slicing](./ReshapeAndSlicing.md)
  - [x] 2.9 [Aliasing - 混淆](./Aliasing.md)
  - [x] 2.10 [Storage orders(lack of links)](./StorageOrders.md)
  - [x] 2.11 [Alignment issues](./AlignmentIssue.md)
    - [x] 2.11.1 [Explanation of the assertion on unaligned arrays](./UnalignedArrayAssert.md)
    - [x] 2.11.2 [Fixed-size vetorizable Eigen objects](./FixedSizeVectorizable.md)
    - [x] 2.11.3 [Structures Having Eigen Members](./StructHavingEigenMembers.md)
    - [x] 2.11.4 [Using STL Containers with Eigen](./UsingSTLContainersWithEigen.md)
    - [x] 2.11.5 [Passing Eigen objects by value to functions](./PassingByValue.md)
    - [x] 2.11.6 [Compiler making a wrong assumption on stack alignment](./WrongStackAlignment.md)
  - [ ] 2.12 [Catalog of coefficient-wise math functions](./CoeffwiseMathFunctions.md)
  - [x] 2.13 [Quick reference guide](./QuickReferenceGuide.md)
- [ ] 3. Dense linear problems and decompositions
  - [Doing-stupidgrass] 3.1 Linear algebra and decompositions
  - [ ] 3.2 Catalogue of dense decompositions
  - [Doing-tianzhiyi] 3.3 Solving linear least squares systems
  - [x] 3.4 [Inplace matrix decompositions](./InplaceDecomposition.md)
  - [x] 3.5 [Benchmark of dense decompositions](/DenseDecompositionBenchmark.md)
- [x] 4. [Sparse linear algebra] - 稀疏线性代数
  - [x] 4.1 [Sparse matrix manipulations- 稀疏矩阵运算](./SparseMatrixManipulations.md) 
  - [Doing-gaojing8500] 4.2 [Solving Sparse Linear Systems - 求解稀疏线性系统](./SolvingSparseLinearSystems.md)
  - [x] 4.3 [Matrix-free solvers - 任意矩阵求解器](./MatrixfreeSolverExample.md)

## Attention

在对某一节内容进行翻译的时候，请将对应小节的状态更改成*Doing*，完成之后再对状态进行更改，并添加链接。
