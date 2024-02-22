[Title](Appendix-A.md)---
typora-root-url: ..\..\img
---

## 附录 B. LLVM 向量化(Vectorizer) {.unnumbered}

\markright{附录 B}

本章节描述了截至 2020 年 Clang 编译器内部 LLVM 循环向量化的状态。内部循环向量化是将最内层循环中的代码转换为使用跨多个循环迭代的矢量的代码的过程。 SIMD 向量中的每个通道对连续的循环迭代执行独立算术运算。 通常，循环不会处于干净状态，向量化必须猜测和假设缺少的信息并会在运行时检查细节。 如果假设错误，向量化会退回到运行标量循环。 下面的示例突出了一些 LLVM 向量化支持的代码模式。

## 循环次数未知 {.unnumbered .unlisted}

LLVM 循环向量化支持循环次数未知的情况。 在下面的循环中，迭代开始和结束点未知，向量化有一个机制可以向量化不以零开始的循环。 在这个例子中，`n` 可能不是向量宽度的倍数，向量化必须将最后几个迭代作为标量代码执行。 保留循环的标量副本会增加代码大小。

```c++
void bar(float* A, float* B, float K, int start, int end) {
  for (int i = start; i < end; ++i)
    A[i] *= B[i] + K;
}
```

## 指针的运行时检查 {.unnumbered .unlisted}

在下面的示例中，如果指针 A 和 B 指向连续的地址，则向量化代码是非法的，因为某些 A 的元素会在从数组 B 读取之前写入。

一些程序员使用 `restrict` 关键字通知编译器指针是不相交的，但在我们的例子中，LLVM 循环向量化无法知道指针 A 和 B 是唯一的。 循环向量化通过放置代码来处理这个循环，该代码在运行时检查数组 A 和 B 是否指向不相交的内存位置。 如果数组 A 和 B 重叠，则执行循环的标量版本。

```c++
void bar(float* A, float* B, float K, int n) {
  for (int i = 0; i < n; ++i)
    A[i] *= B[i] + K;
}
```

## 归约 {.unnumbered .unlisted}

在这个例子中，求和变量被循环的连续迭代使用。 通常，这会阻止向量化，但向量化可以检测到 `sum` 是一个归约变量。 变量 `sum` 成为一个整数向量，在循环结束时，数组的元素被相加以创建正确的结果。 LLVM 循环向量化支持许多不同的归约操作，例如加、乘、异或、与和或。

```c++
int foo(int *A, int n) {
  unsigned sum = 0;
  for (int i = 0; i < n; ++i)
    sum += A[i] + 5;
return sum;
}
```

使用 `-ffast-math` 时，LLVM 循环向量化支持浮点归约操作。

## 归纳 {.unnumbered .unlisted}

在这个例子中，归纳变量 i 的值被保存到一个数组中。 LLVM 循环向量化知道如何向量化归纳变量。

```c++
void bar(float* A, int n) {
  for (int i = 0; i < n; ++i)
    A[i] = i;
}
```

## If 转换 {.unnumbered .unlisted}

LLVM 循环向量器能够“平铺”代码中的 IF 语句并生成单个指令流。 向量化支持最内层循环中的任何控制流。 最内层循环可能包含复杂的 IF、ELSE 甚至 GOTO 的嵌套。

```c++
int foo(int *A, int *B, int n) {
  unsigned sum = 0;
  for (int i = 0; i < n; ++i)
    if (A[i] > B[i])
      sum += A[i] + 5;
  return sum;
}
```

 ## 指针归纳变量 {.unnumbered .unlisted}

此示例使用标准 C++ 库中的 `std::accumulate` 函数。 此循环使用 C++ 迭代器，它们是指针，而不是整数索引。 LLVM 循环向量化器检测指针归纳变量并可以向量化此循环。 此功能很重要，因为许多 C++ 程序使用迭代器。

```c++
int baz(int *A, int n) {
  return std::accumulate(A, A + n, 0);
}
```

## 反向迭代器 {.unnumbered .unlisted}

LLVM 循环向量化器可以向量化反向计数的循环。

```c++
int foo(int *A, int n) {
  for (int i = n; i > 0; --i)
    A[i] +=1;
}
```

## Scatter / Gather {.unnumbered .unlisted}

LLVM 循环向量化器可以向量化成为分散/收集内存的标量指令序列的代码。

```c++
int foo(int * A, int * B, int n) {
  for (intptr_t i = 0; i < n; ++i)
    A[i] += B[i * 4];
}
```

在许多情况下，成本模型会认为这种转换是无利可图的。

## 混合类型的向量化 {.unnumbered .unlisted}

LLVM 循环向量化器可以向量化具有混合类型的程序。 向量化器成本模型可以估计类型转换的成本并决定向量化是否有利可图。

```c++
int foo(int *A, char *B, int n) {
  for (int i = 0; i < n; ++i)
    A[i] += 4 * B[i];
}
```

## 函数调用的向量化 {.unnumbered .unlisted}

LLVM 循环向量化器可以向量化内在数学函数。 有关这些功能的列表，请参见下表。

```bash
pow        exp        exp2
sin        cos        sqrt
log        log2       log10
fabs       floor      ceil
fma        trunc      nearbyint
fmuladd
```

## 向量化过程中的部分展开 {.unnumbered .unlisted}

现代处理器具有多个执行单元，只有包含高度并行性的程序才能充分利用机器的全部宽度。 LLVM 循环向量化器通过执行循环的部分展开来增加指令级并行度（ILP）。

在下面的示例中，整个数组累积到变量 `sum` 中。 这是低效的，因为处理器只能使用单个执行端口。 通过展开代码，循环向量化器允许同时使用两个或多个执行端口。

```c++
int foo(int *A, int n) {
  unsigned sum = 0;
  for (int i = 0; i < n; ++i)
    sum += A[i];
  return sum;
}
```

LLVM 循环向量化器使用成本模型来决定何时展开循环是有利的。 展开循环的决定取决于寄存器压力和生成的代码大小。

## SLP 向量化 {.unnumbered .unlisted}

SLP（超字级并行Superword-Level Parallelism）向量化器试图将多个标量操作粘合在一起形成向量操作。 它自下而上地处理代码，跨越基本块，以寻找要组合的标量。 SLP 向量化的目标是将类似的独立指令组合成向量指令。 内存访问、算术运算、比较运算都可以使用这种技术进行向量化。 例如，以下函数对其输入 (a1, b1) 和 (a2, b2) 执行非常相似的操作。 基本块向量化器可以将以下函数组合成向量运算。

```c++
void foo(int a1, int a2, int b1, int b2, int *A) {
  A[0] = a1*(a1 + b1);
  A[1] = a2*(a2 + b2);
  A[2] = a1*(a1 + b1);
  A[3] = a2*(a2 + b2);
}
```
