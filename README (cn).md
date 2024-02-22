# perf-book

这是一本名为《现代 CPU 上的性能分析与调优》的书籍的源文件存储库，由 Denis Bakhvalov 等人编写。

**第二版正在进行中！** 计划的更改在谷歌[文档](https://docs.google.com/document/d/1tr2qRDe72VSBYypIANYjJLM_zCdPB6S9m4LmXsQb0vQ/edit?usp=sharing)中进行了概述。计划中的新目录在 [new_toc.md](new_toc.md) 中。

我的目标是从行业中的所有最优秀的专家那里积累尽可能多的知识。当然，我会与你分享这些知识。欢迎贡献。

# 贡献

有许多方式可以帮助。
- 您可以撰写您是专家的主题的一节或几节。但在开始之前请通知我。
- 欢迎进行小的改进，无需事先批准，只需打开一个新的 PR。
- 欢迎提出新内容的想法。
- 需要具有各种背景的审阅者。

查看 [discussions](https://github.com/dendibakh/perf-book/discussions) 页面开始。

有关如何添加图像、表格、代码清单等的示例，请参阅 [how-to.md](how-to.md)。

# 构建书籍（pdf）

目前，仅在 Windows 和 Linux 上构建 PDF 可以正常工作。MacOS 需要构建一些组件（例如 pandoc-crossref）。

要求：

 * Python3。安装 natsort 模块：`pip install natsort`。
 * [pandoc](https://pandoc.org/installing.html) - 安装 [版本 2.9](https://github.com/jgm/pandoc/releases/tag/2.9.2.1)。
 * 安装 pandoc 过滤器：`pip install pandoc-fignos pandoc-tablenos`。
 * 安装 `pandoc-crossref`。这需要手动安装。我只是从 [这里](https://github.com/lierdakil/pandoc-crossref/releases/tag/v0.3.6.4) 下载了二进制文件，并将其复制到与 `pandoc-fignos` 相同的位置。
 * [MiKTeX](https://miktex.org/download) - 对于自动包安装选 `Yes`

运行：
```bash
# Linux bash
python export_book.py && pdflatex book.tex && bibtex book && pdflatex book.tex && pdflatex book.tex

# Windows Powershell
function Run-Block-With-Error($block) {
    $ErrorActionPreference="Stop"
    Invoke-Command -ScriptBlock $block
}
Run-Block-With-Error {python.exe export_book.py; pdflatex book.tex; bibtex book; pdflatex book.tex; pdflatex book.tex}
```

结果，将生成 `book.pdf`。由于安装所需的数据包，第一次编译可能会比较慢。

## 许可证

[创作共用零版本 1.0 通用](LICENSE)