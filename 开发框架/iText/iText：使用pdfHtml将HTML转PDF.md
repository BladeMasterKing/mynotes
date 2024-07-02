# iText：使用pdfHTML将HTML转换为PDF



## 介绍

`pdfHTML`是 `iText 7` 的一个附加组件，将 `HTML` 转换为 `PDF`。在旧版的 `iText 5` 中一直在使用已过时的 `HTMLWorker` 类或旧的 `XML Worker` 附加组件。

`HTMLWorker` 类在多年前就已被弃用。`HTMLWorker` 的目标是将小型、简单的 `HTML` 片段转换为 `iText` 对象。它从未被设计成将完整的 `HTML` 页面转换为 `PDF` ，但许多开发人员却试图这样使用它。这导致了很多问题，因为 `HTMLWorker` 不支持每个 `HTML` 标签，不解析 `CSS` 文件等。为了避免这种沮丧，`HTMLWorker` 已从 `iText` 的最新版本中删除。

2011年，`iText Group` 发布了 `XML Worker` 作为一个通用的 `XML` 到 `PDF` 工具，建立在`iText 5` 之上。默认实现将 `XHTML`（数据）和 `CSS`（样式）转换为 `PDF`，将 `HTML` 标签（如   `<p>`、`<img>`和`<li>`）映射到`iText 5`对象（如`Paragraph`、`Image`和`ListItem`）。我们不知道有没有使用 `XML Worker` 用于其他 `XML` 格式的任何实现，但许多开发人员将 `XML Worker` 与 `jsoup` 结合使用作为 `HTML2PDF` 转换器。

但 `XML Worker` 不是 `URL2PDF` 工具。`XML Worker` 期望可预测的 `HTML`，专门用于将 `HTML` 转换为 `PDF`。常见用例是创建发票。开发人员选择创建一个简单的 `HTML` 模板来定义文档的结构，并使用一些CSS来定义样式，而不是在Java或C#中编程发票的设计。然后，他们使用数据填充 `HTML`，并使用 `XML Worker` 将发票创建为PDF文档，丢弃原始的 `HTML`。我们将在第4章中更详细地讨论这种用例，使用 `XSLT` 将 `XML` 转换为`HTML`内存中，然后使用pdfHTML附加组件将其转换为PDF。

当最初创建iText 5时，它被设计为尽可能快地生成PDF，将页面刷新到OutputStream时完成。一些设计选择在2000年iText首次发布时是完全合理的，16年后仍然存在于iText 5中。不幸的是，其中一些选择使得将XML Worker的功能扩展到许多开发人员期望的质量水平变得非常困难，甚至不可能。如果我们真的想要创建一个出色的HTML到PDF转换器，我们将不得不从头开始重写iText。我们也做到了。

2016年，我们发布了iText 7，这是iText的全新版本，不再与旧版本兼容，但是考虑到pdfHTML而创建的。大量的工作花在了新的Renderer框架上。当使用iText 7创建文档时，会构建一个渲染器及其子渲染器的树。通过遍历该树来创建布局，这种方法在处理HTML到PDF转换时更加合适。iText对象被完全重新设计，以更好地匹配HTML标签，并允许以“CSS方式”设置样式。

例如：在iText 5中，您需要一个PdfPTable和一个PdfPCell对象来创建表格及其单元格。如果您希望每个单元格包含不同于默认字体的文本，则需要为每个单独的单元格的内容设置该字体。在iText 7中，您有一个Table和Cell对象，当您为整个表格设置不同的字体时，该字体将被继承为每个单元格的默认字体。这在架构设计方面是一个重大进步，特别是如果目标是将HTML转换为PDF。

但是让我们不要沉湎于过去，让我们看看pdfHTML能为我们做些什么。在第一章中，我们将研究convertToPdf()/ConvertToPdf()方法的不同变体，并发现转换器是如何配置的。
