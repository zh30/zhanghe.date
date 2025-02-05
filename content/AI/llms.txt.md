---
title: llms.txt 文件详解
date: 2025-02-05
---
大型语言模型 (LLM) 越来越依赖网站信息，但有一个关键限制：上下文窗口太小，无法完整处理大多数网站。将包含导航、广告和 JavaScript 的复杂 HTML 页面转换成 LLM 友好的纯文本既困难又难以精确。

网站既服务于人类读者，也服务于 LLM，但 LLM 更受益于更简洁、专家级的信息，这些信息集中在一个易于访问的地方。这对于开发环境之类的用例尤其重要，因为 LLM 需要快速访问编程文档和 API。

##  介绍 llms.txt 文件

建议在网站上添加一个`/llms.txt` Markdown 文件，以提供 LLM 友好的内容。这个文件提供了简要的背景信息、指导和链接到详细的 Markdown 文件。
`/llms.txt` Markdown 文件既方便人类阅读，也方便 LLM 阅读，但它也采用精确的格式，允许使用固定的处理方法（例如，经典的编程技术，如解析器和正则表达式）。

网站上包含可能对 LLM 有用的信息的页面，在与原始页面相同的 URL 上提供一个干净的 Markdown 版本，但使用 `.md` 扩展名。（没有文件名 URL 的情况下，请附加 `index.html.md`。）
FastHTML 项目遵循这两个建议来构建其文档。例如，这里有 FastHTML 文档的 `/llms.txt`。这里有一个普通 HTML 文档页面，以及具有相同 URL 但 `.md` 扩展名的示例。

此建议不包含任何特定处理 `/llms.txt` 文件的建议，因为它取决于应用程序。例如，FastHTML 项目选择将 `/llms.txt` 自动扩展为两个 Markdown 文件，其中包含链接 URL 的内容，并使用适合 LLM（例如 Claude）使用的基于 XML 的结构。这两个文件是：`/llms-ctx.txt`（不包含可选 URL）和 `/llms-ctx-full.txt`（包含可选 URL）。它们使用 `/llms_txt2ctx` 命令行应用程序创建，FastHTML 文档包含有关如何使用的用户指南。

`/llms.txt` 文件的多功能性意味着它可以用于许多用途——从帮助开发人员了解软件文档，到让企业概述其结构，甚至为利益相关者分解复杂的立法。对于个人网站，它们同样有用，可以帮助回答有关个人简历的问题；对于电商网站，可以解释产品和政策；对于学校和大学，可以快速访问课程信息和资源。
注意，所有 nbdev 项目现在默认创建所有页面的 `.md` 版本。所有使用 nbdev 的 Answer.AI 和 fast.ai 软件项目都已使用此功能重新生成了文档。例如，请参阅 fastcore 的文档模块的 Markdown 版本。

## 格式

目前，语言模型最广泛、最易理解的格式是 Markdown。只需显示关键 Markdown 文件的位置即可迈出重要一步。提供一些基本结构有助于语言模型找到所需信息来源。

`/llms.txt` 文件的独特之处在于，它使用 Markdown 来组织信息，而不是经典的结构化格式，例如 XML。原因是我们预计许多这些文件都将被语言模型和代理程序读取。话虽如此，`/llms.txt` 文件中的信息遵循特定格式，可以使用标准的基于编程的工具进行读取。

`/llms.txt` 文件规范适用于位于网站根路径 `/llms.txt` （或可选的子路径）的文件。遵循此规范的文件包含以下部分，按特定顺序使用 Markdown：


*  一个包含项目或网站名称的 H1 标题。这是唯一必需的部分。
*  一个包含项目简要摘要的区块引用，其中包含理解文件其余部分所需的关键信息。
*  零个或多个任何类型（除标题外）的 Markdown 部分（例如段落、列表等），包含有关项目及其如何解释所提供文件的更多详细信息。
*  零个或多个由 H2 标题分隔的 Markdown 部分，包含“文件列表”URL，其中包含更多详细信息。

每个“文件列表”都是一个 Markdown 列表，包含一个必需的 Markdown 超链接 [名称](URL)，然后可选地添加一个冒号和有关该文件的注释。

示例：
```markdown
# 标题

> 可选描述在此处

可选详细信息在此处

## 部分名称

- [链接标题](https://link_url): 可选链接详细信息

## 可选

- [链接标题](https://link_url)
```
注意，“可选”部分具有特殊含义——如果包含它，则在需要更短的上下文的情况下，可以跳过其中提供的 URL。将其用于通常可以跳过的辅助信息。

## 现有标准

`/llms.txt` 旨在与当前 Web 标准共存。虽然站点地图列出了搜索引擎的所有页面，但 `/llms.txt` 为 LLM 提供了精选的概述。它可以通过提供允许内容的上下文来补充 robots.txt。该文件还可以引用网站上使用的结构化数据标记，帮助 LLM 了解如何在上下文中解释这些信息。

此文件路径标准化方法与 `/robots.txt` 和 `/sitemap.xml` 的方法相同。`/robots.txt` 和 `/llms.txt` 的用途不同——`/robots.txt` 通常用于告知自动化工具网站访问被认为是可以接受的，例如用于搜索索引机器人。另一方面，`/llms.txt` 信息通常会在用户明确请求有关某个主题的信息时按需使用，例如在项目中包含编码库的文档时，或在具有搜索功能的聊天机器人中请求信息时。我们预期 `/llms.txt` 主要用于推理，即在用户寻求帮助时，而不是用于训练。不过，如果 `/llms.txt` 的使用变得广泛，未来的训练运行也可能利用 `/llms.txt` 文件中的信息。

`/sitemap.xml` 是网站上所有可索引的人类可读信息的列表。这并不是 `/llms.txt` 的替代品，因为它：

* 通常不会列出页面的 LLM 可读版本。
* 不包含指向外部网站的 URL，即使它们可能有助于理解信息。
* 通常会涵盖文档，这些文档的总和太大，无法放入 LLM 上下文窗口中，并且会包含很多不必要的信息来理解网站。

## 示例

这是 `/llms.txt` 的示例，在本例中是 FastHTML 项目所用文件（缩减版）（另请参阅完整版本：
```markdown
# FastHTML

> FastHTML 是一个 Python 库，它将 Starlette、Uvicorn、HTMX 和 fastcore 的`FT`（“FastTags”）整合到一个用于创建服务器渲染超媒体应用程序的库中。

重要说明：

- 尽管其 API 的一部分受到 FastAPI 的启发，但它与 FastAPI 语法不兼容，并且目标不是创建 API 服务。
- FastHTML 与原生 JS Web 组件和任何普通 JS 库兼容，但不兼容 React、Vue 或 Svelte。

## 文档

- [FastHTML 快速入门](https://docs.fastht.ml/path/quickstart.html.md)：许多 FastHTML 功能的简要概述
- [HTMX 参考](https://raw.githubusercontent.com/path/reference.md)：所有 HTMX 属性、CSS 类、标头、事件、扩展、JS 库方法和配置选项的简要说明

## 示例

- [待办事项列表应用程序](https://raw.githubusercontent.com/path/adv_app.py)：FastHTML 和 HTMX 模式惯用示例的完整 CRUD 应用的详细演练。

## 可选

- [Starlette 完整文档](https://gist.githubusercontent.com/path/starlette-sml.md)：对 FastHTML 开发有用的 Starlette 文档子集。
```


## 创建有效 `/llms.txt` 文件的建议

使用简洁明了的语言。
链接到资源时，包含简明扼要的描述。
避免模棱两可的术语或未解释的术语。
运行一个工具，将 `/llms.txt` 文件扩展到 LLM 上下文文件，并测试多种语言模型，看看它们是否能够回答有关您内容的问题。

## 文件夹

这里列出了一些包含 `/llms.txt` 文件的网站目录：

`/llmstxt.site`
`/directory.llmstxt.cloud`

