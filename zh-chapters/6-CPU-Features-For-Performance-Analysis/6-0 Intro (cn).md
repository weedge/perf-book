# CPU 特性用于性能分析 {#sec:PmuChapter}

性能分析的最终目标是识别性能瓶颈并定位与之相关的代码部分。不幸的是，没有预先确定的步骤可以遵循，因此可以采取许多不同的方法。

通常，对应用程序进行分析可以快速洞察应用程序的热点。有时这是开发人员需要做的所有事情，以修复性能效率问题。特别是高层次的性能问题通常可以通过分析来揭示。例如，考虑一种情况，当你刚刚对应用程序中的函数`foo`进行了更改，突然看到性能显著下降。因此，你决定对应用程序进行分析。根据你对应用程序的心理模型，你预计`foo`是一个冷函数，它不会出现在热函数的前10名列表中。但是当你打开分析结果时，你看到它比以前消耗了更多的时间。你很快意识到你在代码中犯了一个错误，并修复了它。如果所有性能工程问题都那么容易解决，这本书就不会存在了。

当你开始努力从应用程序中挤出最后一点性能时，最基本的热点列表是不够的。除非你有一个水晶球或者对整个CPU有一个准确的模型，否则你需要额外的支持来理解性能瓶颈是什么。然而，在本章中使用所呈现的信息之前，请确保你正在尝试优化的应用程序没有重大的性能缺陷。因为如果它有，使用CPU性能监控特性进行低级调整是没有意义的。这可能会误导你的方向，而不是修复真正的高层次性能问题，你将只是在调整糟糕的代码，这纯粹是浪费时间。

一些开发人员依赖于他们的直觉，并进行随机实验，尝试强制各种编译器优化，比如循环展开、向量化、内联等等。确实，有时你可以很幸运，并从你的同事那里得到一些赞美，甚至可能在你的团队中声称一个非官方的性能大师的头衔。但通常，你需要有很好的直觉和运气。在这本书中，我们不教你如何变得幸运。相反，我们展示了在实践中被证明有效的方法。

现代CPU不断获得新的特性，以不同的方式增强性能分析。使用这些特性可以大大简化发现低级问题，如缓存未命中、分支预测错误等。在本章中，我们将看看现代CPU上可用的一些硬件性能监控功能。不同厂商的处理器不一定具有相同的功能集。在本章中，我们将重点关注Intel、AMD和ARM处理器中可用的性能监控功能。RISC-V生态系统还没有成熟的性能监控基础设施，所以我们在这里不会涵盖它。

* **自顶向下微架构分析** (TMA) 方法论，讨论在 [@sec:TMA] 中。这是一种强大的技术，用于识别程序对CPU微架构的无效使用。它描述了工作负载的瓶颈，并允许定位源代码中发生瓶颈的确切位置。它抽象了CPU微架构的复杂性，即使对于没有经验的开发者来说也相对容易使用。
* **最后分支记录** (LBR)，讨论在 [@sec:lbr] 中。这是一种机制，它在执行程序的同时连续记录最近的分支结果。它用于收集调用栈，识别热分支，计算单个分支的误预测率等。
* **基于处理器事件的采样** (PEBS)，讨论在 [@sec:secPEBS] 中。这是一个增强采样的特性。其主要好处包括：降低采样的开销；并提供“精确事件”功能，这使得能够精确定位导致特定性能事件的确切指令。
* **Intel处理器跟踪** (PT)，在附录D中讨论。它是一个记录和重构程序执行的功能，每个指令都有一个时间戳。它的主要用途是事后分析和根除性能故障。

Intel PT特性在附录D中介绍。Intel PT本应成为性能分析的“终极解决方案”。由于其低运行时开销，它是一个非常强大的分析特性。但事实证明，它在性能工程师中并不受欢迎。部分原因是工具支持不成熟，部分原因是在许多情况下它过于复杂，而且使用采样分析器更容易。此外，它产生大量数据，对于长时间运行的工作负载来说并不实用。

上述特性提供了从CPU角度对程序效率的洞察。在下一章中，我们将讨论分析工具如何利用它们提供多种类型的性能分析。