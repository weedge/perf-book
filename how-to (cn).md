## 图片

### 简单图片

```
![40 年微处理器趋势数据。](../img/1/40-years-processor-trend.png){#fig:40YearsProcessorTrend width=90%}
```
在文本中引用它为 `@fig:40YearsProcessorTrend`

### 两张图片并排

```
<div id="fig:BBLayout">
![默认布局](../../img/5/BBLayout_Default.jpg){#fig:BB_default width=30%}
![改进后的布局](../../img/5/BBLayout_Better.jpg){#fig:BB_better width=30%}

代码片段的两种不同布局。
</div>
``` 
在文本中引用它为 `@fig:BB_default`、`@fig:BB_better` 或 `@fig:BBLayout`
    
## 表格

```
--------------------------------------------------------------------------
事件        掩码  事件掩码            描述
编号        值    助记符              
------      ----- --------------------- ---------------------------------------
C0H         00H   INST_RETIRED.         撤回的指令数。 
                     ANY_P

C4H         00H   BR_INST_RETIRED.      撤回的分支指令。
                     ALL_BRANCHES                  
--------------------------------------------------------------------------

表格：Skylake 性能事件的编码示例。{@tbl:perf_count}
```

在文本中引用它为 `{@tbl:perf_count}`

## 论文/书籍引用

要引用一本书籍，首先以 BibTeX 格式下载其引用，例如：

```
@Article{Mytkowicz09,
  author     = {Mytkowicz, Todd and Diwan, Amer and Hauswirth, Matthias and Sweeney, Peter F.},
  title      = {Producing Wrong Data without Doing Anything Obviously Wrong!},
  journal    = {SIGPLAN Not.},
  year       = {2009},
  volume     = {44},
  number     = {3},
  pages      = {265–276},
  month      = mar,
  issn       = {0362-1340},
  address    = {New York, NY, USA},
  doi        = {10.1145/1508284.1508275},
  issue_date = {February 2009},
  keywords   = {measurement, bias, performance},
  numpages   = {12},
  publisher  = {Association for Computing Machinery},
  url        = {https://doi.org/10.1145/1508284.1508275},
}
```

在文本中以 `[@Mytkowicz]` 引用论文。

并非所有引用中的字段都是必需的，但至少需要 `title`、`organization`、`year`、`url`。

## 代码清单

```
清单：a.c

~~~~ {#lst:optReport .cpp .numberLines}
void foo(float* __restrict__ a, 
         float* __restrict__ b, 
         float* __restrict__ c,
         unsigned N) {
  for (unsigned i = 1; i < N; i++) {
    a[i] = c[i-1]; // value is carried over from previous iteration
    c[i] = b[i];
  }
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

在文本中引用它为 `[@lst:optReport]`

对于小片段或命令行脚本，您也可以使用更简单的版本：

```
```$ perf record -- ./ls```
```

## 脚注

```
[^1]: Profiling(wikipedia) - [https://en.wikipedia.org/wiki/Profiling_(computer_programming)](https://en.wikipedia.org/wiki/Profiling_(computer_programming)).
```

在相同的 `*.md` 文件中引用脚注为 `[^1]`

## 节的引用

要从另一章引用书中的一节，请首先向您要引用的节的标题添加一个标签，例如：

```
# 性能分析 {#sec:sec1}
```

在文本中以 `[@sec:sec1]` 引用该节

## 数学公式

下面是两个数学公式的示例：一个简单的和一个更复杂的。您可以查看它们的呈现方式。

```
$$
IPC = \frac{INST\_RETIRED.ANY}{CPU\_CLK\_UNHALTED.THREAD}
$$

$$
\begin{aligned}
\textrm{Peak FLOPS} =& \textrm{ 8 (number of logical cores)}~\times~\frac{\textrm{256 (AVX bit width)}}{\textrm{32 bit (size of float)}} ~ \times ~ \\
& \textrm{ 2 (FMA)} \times ~ \textrm{3.8 GHz (Max Turbo Frequency)} \\
& = \textrm{486.4 GFLOPs}
\end{aligned}
$$
```

## 引用

```
> "在摩尔定律后期，使代码运行快速，特别是将其定制到其运行的硬件上，将变得越来越重要。" [@Leisersoneaam9744]
```