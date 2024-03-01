## 编译器内联函数 {#sec:secIntrinsics}

有些类型的应用程序有值得大力调整的热点。然而，编译器在这些热点生成的代码并不总是符合我们的期望。例如，程序在循环中进行一些计算，而编译器以次优的方式向量化。这通常涉及到一些棘手或特殊的算法，我们可以为这些算法设计出更好的指令序列。使用C和C++语言的标准结构让编译器生成所需的汇编代码可能非常困难，甚至是不可能的。

幸运的是，有可能在不编写低级汇编语言的情况下，强制编译器生成特定的汇编指令。为了实现这一点，可以使用编译器内联函数，它们反过来被翻译成特定的汇编指令。内联函数提供了与使用内联汇编相同的好处，但同时也提高了代码的可读性，允许编译器类型检查，协助指令调度，并有助于减少调试。在[@lst:Intrinsics]中的示例展示了如何通过编译器内联函数（函数`bar`）来编码函数`foo`中的相同循环。

代码清单:编译器内部函数

~~~~ {#lst:Intrinsics .cpp .numberLines}
void foo(float *a, float *b, float *c, unsigned N) {
  for (unsigned i = 0; i < N; i++)
    c[i] = a[i] + b[i]; 
}

#include <xmmintrin.h>

void bar(float *a, float *b, float *c, unsigned N) {
  __m128 rA, rB, rC;
  int i = 0;
  for (; i + 3 < N; i += 4){
    rA = _mm_load_ps(&a[i]);
    rB = _mm_load_ps(&b[i]);
    rC = _mm_add_ps(rA,rB);
    _mm_store_ps(&c[i], rC);
  }
  for (; i < N; i++) // remainder
    c[i] = a[i] + b[i];
}
~~~~

当编译为 SSE 目标时，`foo` 和 `bar` 都将生成类似的汇编指令。但是，有几个注意事项。首先，当依赖自动矢量化时，编译器会插入所有必要的运行时检查。例如，它将确保有足够的元素来填充向量执行单元。其次，函数 `foo` 将有一个处理循环剩余部分的标量版本作为备用。最后，大多数向量内部函数假设数据是对齐的，因此为 `bar` 生成 `movaps`（对齐加载），而为 `foo` 生成 `movups`（未对齐加载）。考虑到这一点，开发人员使用编译器内部函数时必须自己注意安全方面。

使用非可移植的平台特定内部函数编写代码时，开发人员还应该为其他架构提供备选项。有关英特尔平台的所有可用内部函数列表，请参见此 参考: [https://software.intel.com/sites/landingpage/IntrinsicsGuide/](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)[^11]。

### 内部函数的包装库

ISPC 的一次编写，多目标运行模式很有吸引力。然而，我们可能希望更紧密地集成到 C++ 程序中，例如与模板互操作，或者避免单独的构建步骤并使用相同的编译器。相反，内部函数提供更多的控制，但开发成本更高。

我们可以结合两者的优势并避免这些缺点，使用所谓的嵌入式领域特定语言，其中向量操作表示为普通的 C++ 函数。您可以将这些函数视为“可移植的内部函数(portable intrinsics)”，例如 `Add` 或 `LoadU`。即使多次编译您的代码（每个指令集一次），也可以在普通 C++ 库中完成，方法是使用预处理器在不同的编译器设置下“重复”您的代码，但位于独特的命名空间中。一个例子是之前提到的 Highway 库，[^12] 它只要求 C++11 标准。

与 ISPC 一样，Highway 也支持检测最佳可用指令集，这些指令集分为“簇(clusters)”，在 x86 上对应于 Intel Core (S-SSE3)、Nehalem (SSE4.2)、Haswell (AVX2)、Skylake (AVX-512) 或 Icelake/Zen4 (AVX-512 with extensions)。然后它从相应命名空间调用您的代码。

与内部函数不同，代码保持可读性（每个函数没有前缀/后缀）和可移植性。

代码清单:对数组元素求和的高速版本。

~~~~ {#lst:HWY_code .cpp}
#include <hwy/highway.h>

float calcSum(const float* HWY_RESTRICT array, size_t count) {
  const ScalableTag<float> d;  // type descriptor; no actual data
  auto sum = Zero(d);
  size_t i = 0;
  for (; i + Lanes(d) <= count; i += Lanes(d)) {
    sum = Add(sum, LoadU(d, array + i));
  }
  sum = Add(sum, MaskedLoad(FirstN(d, count - i), d, array + i));
  return ReduceSum(d, sum);
}
~~~~

注意循环处理向量大小`Lanes(d)`的倍数后余数的显式处理。虽然这更加冗长，但它使实际发生的事情变得可见，并允许进行优化，例如重叠最后一个向量而不是依赖于`MaskedLoad`，或者当`count`已知是向量大小时完全跳过余数。

Highway支持超过200种操作，可以分为以下几类：

* 初始化(Initialization)
* 获取/设置车道(Getting/setting lanes)
* 获取/设置块(Getting/setting blocks)
* 打印(Printing)
* 元组(Tuples)
* 算术(Arithmetic)
* 逻辑(Logical)
* 掩码(Masks)
* 比较(Comparisons)
* 内存(Memory)
* 缓存控制(Cache control)
* 类型转换(Type conversion)
* 组合(Combine)
* 洗牌/排列(Swizzle/permute)
* 在128位块内进行洗牌(Swizzling within 128-bit blocks)
* 归约(Reductions)
* 加密(Crypto)

有关操作的完整列表，请参见其文档[^13]和[常见问题解答](https://github.com/google/highway/blob/master/g3doc/faq.md)。您还可以在在线[Compiler Explorer](https://gcc.godbolt.org/z/zP7MYe9Yf)中尝试它。

其他库包括Eigen、nsimd、SIMDe、VCL和xsimd。请注意，从Vc库开始的C++标准化工作导致了std::experimental::simd，但它提供了非常有限的操作集，并且截至本文撰写时仅在GCC 11编译器上支持。

[^11]: 英特尔内部函数指南 - [https://software.intel.com/sites/landingpage/IntrinsicsGuide/](https://software.intel.com/sites/landingpage/IntrinsicsGuide/)。
[^12]: Highway 库: [https://github.com/google/highway](https://github.com/google/highway)
[^13]: Highway 快速参考 - [https://github.com/google/highway/blob/master/g3doc/quick_reference.md](https://github.com/google/highway/blob/master/g3doc/quick_reference.md)
