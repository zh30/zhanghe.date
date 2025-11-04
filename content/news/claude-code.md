---
title: 效率神器 Claude Code：让你的代码飞起来！
date: 2025-02-25
---

Claude Code 是个啥？简单说，它就是个能在你终端里跑的智能编码助手，贼懂你的代码，能用大白话帮你快速写代码。它直接和你的开发环境无缝衔接，不用搭服务器，也不用搞那些复杂的配置，直接就能用，超省事儿！

Claude Code 主要能干这些：

- 修改代码文件，帮你修 bug，整个项目都能搞定。
- 回答你关于代码结构和逻辑的各种问题，就像有个代码大神在你身边。
- 执行测试、代码检查啥的，还能帮你修复问题。
- 搜索 Git 历史记录，解决合并冲突，创建提交和 PR，一条龙服务！

## 开始之前，先看看你的电脑够不够格

### 系统要求

- 操作系统：macOS 10.15+, Ubuntu 20.04+/Debian 10+, Windows (用 WSL 也行)。
- 硬件：最少 4GB 内存。
- 软件：
- Node.js 18+
- git 2.23+ (可选)
- GitHub 或 GitLab CLI，如果你要用 PR 功能 (可选)
- ripgrep (rg)，能让你更快地搜索文件 (可选)

- 网络：要联网才能验证身份和进行 AI 处理。

### 安装和身份验证

安装完之后记得验证身份才能用哦！

## 核心功能和工作流程

Claude Code 直接在你终端里干活，能理解你的项目，直接帮你搞定事情。不用手动添加文件，它自己会探索你的代码库。默认用的是 claude-3-7-sonnet-20250219 模型。

### 安全和隐私至上

你的代码安全是最重要的！Claude Code 的架构保证：

- **直接 API 连接**：你的请求直接发给 Anthropic 的 API，没有中间服务器。
- **在哪工作就在哪用**：直接在你终端里运行。
- **理解上下文**：能记住你的整个项目结构。
- **直接操作**：能真的帮你改文件，创建提交。

从问题到解决方案，只要几秒钟！

## 初始化你的项目

第一次用的话，推荐你：

- 用 `claude` 命令启动 Claude Code。
- 试试像 `summarize this project` 这样的简单命令。
- 用 `/init` 命令生成一个 `CLAUDE.md` 项目指南。
- 让 Claude 帮你把 `CLAUDE.md` 文件提交到你的代码仓库。

## Claude Code 常用姿势

Claude Code 直接在你终端里干活，能理解你的项目，直接帮你搞定事情。不用手动添加文件，它自己会探索你的代码库。

- **理解你不熟悉的代码**
- **自动化 Git 操作**
- **智能编辑代码**
- **测试和调试你的代码**

### 让 Claude 想得更深入

遇到复杂问题，可以明确地要求 Claude 深入思考：

## 用命令来控制 Claude Code

### 命令行 (CLI) 命令

| 命令                            | 描述                               | 例子                                      |
| ------------------------------- | ---------------------------------- | ----------------------------------------- |
| `claude`                        | 启动交互式 REPL                    | `$ claude`                                |
| `claude "query"`                | 启动 REPL，并带初始提示            | `$ claude "explain this project"`         |
| `claude -p "query"`             | 运行一次性查询，然后退出           | `$ claude -p "explain this function"`     |
| `cat file \| claude -p "query"` | 处理管道内容                       | `$ cat logs.txt \| claude -p "explain"`   |
| `claude config`                 | 配置设置                           | `$ claude config set --global theme dark` |
| `claude update`                 | 更新到最新版本                     | `$ claude update`                         |
| `claude mcp`                    | 配置 Model Context Protocol 服务器 | `$ claude mcp add pyright_lsp`            |

CLI 参数：

- `--print`: 打印响应，不进入交互模式
- `--verbose`: 启用详细日志
- `--dangerously-skip-permissions`: 跳过权限提示（仅在没有互联网连接的 Docker 容器中使用）

### 斜杠命令

| 命令              | 目的                                                  |
| ----------------- | ----------------------------------------------------- |
| `/bug`            | 报告 Bug (发送对话到 Anthropic)                       |
| `/clear`          | 清除对话历史记录                                      |
| `/compact`        | 压缩对话，节省上下文空间                              |
| `/config`         | 查看/修改配置                                         |
| `/cost`           | 显示 token 使用统计信息                               |
| `/doctor`         | 检查 Claude Code 安装的健康状况                       |
| `/help`           | 获取用法帮助                                          |
| `/init`           | 使用 CLAUDE.md 指南初始化项目                         |
| `/login`          | 切换 Anthropic 帐户                                   |
| `/logout`         | 注销 Anthropic 帐户                                   |
| `/pr_comments`    | 查看拉取请求评论                                      |
| `/review`         | 请求代码审查                                          |
| `/terminal-setup` | 为换行安装 Shift+Enter 键绑定 (仅限 iTerm2 和 VSCode) |

## 管理权限和安全

Claude Code 使用分层权限系统来平衡能力和安全：

| 工具类型  | 例子               | 需要批准？ | “是，不再询问”行为           |
| --------- | ------------------ | ---------- | ---------------------------- |
| 只读      | 文件读取，LS, Grep | 否         | N/A                          |
| Bash 命令 | Shell 执行         | 是         | 永久地针对每个项目目录和命令 |
| 文件修改  | 编辑/写入文件      | 是         | 直到会话结束                 |

### Claude Code 可用的工具

| 工具             | 描述                                     | 需要权限？ |
| ---------------- | ---------------------------------------- | ---------- |
| AgentTool        | 运行一个子代理来处理复杂的、多步骤的任务 | 否         |
| BashTool         | 在你的环境中执行 shell 命令              | 是         |
| GlobTool         | 根据模式匹配查找文件                     | 否         |
| GrepTool         | 在文件内容中搜索模式                     | 否         |
| LSTool           | 列出文件和目录                           | 否         |
| FileReadTool     | 读取文件的内容                           | 否         |
| FileEditTool     | 对特定文件进行有针对性的编辑             | 是         |
| FileWriteTool    | 创建或覆盖文件                           | 是         |
| NotebookReadTool | 读取和显示 Jupyter notebook 内容         | 否         |
| NotebookEditTool | 修改 Jupyter notebook 单元格             | 是         |

### 防范提示注入

提示注入是一种攻击者试图通过插入恶意文本来覆盖或操纵 AI 助手的指令的技术。Claude Code 包含多种针对这些攻击的保护措施：

- **权限系统**：敏感操作需要明确批准。
- **上下文感知分析**：通过分析完整的请求来检测潜在的有害指令。
- **输入清理**：通过处理用户输入来防止命令注入。
- **命令黑名单**：阻止从网络上获取任意内容的危险命令，例如 curl 和 wget。

使用不受信任的内容的最佳实践：

- 批准之前审查建议的命令
- 避免将不受信任的内容直接管道传输到 Claude
- 验证对关键文件的提议更改
- 使用 `/bug` 报告可疑行为

## 配置网络访问

Claude Code 需要访问：

- api.anthropic.com
- statsig.anthropic.com
- sentry.io

在容器化环境中使用 Claude Code 时，允许列出这些 URL。

## 优化你的终端设置和配置你的环境

Claude Code 在你的终端正确配置时效果最佳。按照这些指南来优化你的体验。

支持的 shell：

- Bash
- Zsh (目前不支持 Fish shell)

### 主题和外观

Claude 无法控制你的终端的主题。这由你的终端应用程序处理。你可以在入门期间或随时通过 `/config` 命令将 Claude Code 的主题与你的终端匹配

### 换行符

你可以使用多种选项将换行符输入到 Claude Code 中：

- **快速转义**：键入 `\` 后按 Enter 以创建新行
- **键盘快捷键**：按 Option+Enter (Meta+Enter) 并进行正确配置

要在你的终端中设置 Option+Enter：

- 对于 Mac Terminal.app：

- 打开“设置” → “配置文件” → “键盘”
- 选中“将 Option 用作 Meta 键”
- 对于 iTerm2 和 VSCode 终端：

- 打开“设置” → “配置文件” → “键”
- 在“常规”下，将左/右 Option 键设置为“Esc+”

iTerm2 和 VSCode 用户的提示：在 Claude Code 中运行 `/terminal-setup` 以自动将 Shift+Enter 配置为更直观的替代方案。

### 通知设置

在正确配置通知的情况下，永远不会错过 Claude 完成任务的时间：

终端响铃通知

启用任务完成时的声音警报：

- 对于 macOS 用户：不要忘记在“系统设置” → “通知” → “[你的终端应用程序]”中启用通知权限。

iTerm 2 系统通知

对于任务完成时 iTerm 2 警报：

- 打开 iTerm 2 首选项
- 导航到“配置文件” → “终端”
- 启用“静音响铃”和“空闲时发送通知”
- 设置你首选的通知延迟

请注意，这些通知特定于 iTerm 2，并且在默认的 macOS 终端中不可用。

### 处理大型输入

当处理大量的代码或长的指令时：

- **避免直接粘贴**：Claude Code 可能难以处理非常长的粘贴内容
- **使用基于文件的工作流程**：将内容写入文件并要求 Claude 读取它
- **注意 VS Code 限制**：VS Code 终端特别容易截断长的粘贴内容

通过配置这些设置，你将使用 Claude Code 创建更顺畅、更高效的工作流程。

## 有效地管理成本

Claude Code 为每次交互消耗 token。典型的使用成本范围为每个开发人员每天 5-10 美元，但在密集使用期间可能超过每小时 100 美元。

### 跟踪你的成本

- 使用 `/cost` 查看当前会话的使用情况
- 查看退出时显示的成本摘要
- 在 Anthropic 控制台中查看历史使用情况

### 设置消费限制

### 减少 token 使用量

- **压缩对话**：当上下文变得很大时使用 `/compact`
- **编写具体的查询**：避免触发不必要的扫描的模糊请求
- **分解复杂的任务**：将大型任务分解为集中的交互
- **清除任务之间的历史记录**：使用 `/clear` 重置上下文

成本可能会因以下因素而显着变化：

- 正在分析的代码库的大小
- 查询的复杂性
- 正在搜索或修改的文件数量
- 对话历史记录的长度
- 压缩对话的频率

## 与第三方 API 一起使用

### 连接到 Amazon Bedrock

如果未启用提示缓存，另请设置：

- 需要标准的 AWS SDK 凭证（例如，`~/.aws/credentials` 或相关的环境变量，例如 `AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`）。请与 Amazon Bedrock 联系以获取提示缓存，以降低成本和提高速率限制。

### 连接到 Google Vertex AI

- 需要通过 google-auth-library 配置的标准 GCP 凭证。为了获得最佳体验，请与 Google 联系以提高速率限制。

## 开发容器参考实现

Claude Code 为需要一致、安全环境的团队提供了一个开发容器配置。这个预配置的 devcontainer 设置与 VS Code 的 Remote - Containers 扩展和类似工具无缝协作。

容器的增强型安全措施（隔离和防火墙规则）允许你运行 `claude --dangerously-skip-permissions` 以绕过无人值守操作的权限提示。我们包含了一个你可以根据你的需求进行自定义的参考实现。

### 主要特点

- **生产就绪的 Node.js**：构建在 Node.js 20 之上，具有必要的开发依赖项
- **安全设计**：自定义防火墙，将网络访问限制为仅必要的服务
- **开发人员友好的工具**：包括 git、带有生产力增强功能的 ZSH、fzf 等
- **无缝 VS Code 集成**：预配置的扩展和优化的设置
- **会话持久性**：在容器重启之间保留命令历史记录和配置
- **无处不在**：与 macOS、Windows 和 Linux 开发环境兼容

### 4 个步骤入门

- 安装 VS Code 和 Remote - Containers 扩展
- 克隆 Claude Code 参考实现存储库
- 在 VS Code 中打开存储库
- 出现提示时，单击“在容器中重新打开”（或使用命令面板：Cmd+Shift+P → “Remote-Containers: 在容器中重新打开”）

### 配置分解

devcontainer 设置由三个主要组件组成：

- `devcontainer.json`：控制容器设置、扩展和卷挂载
- `Dockerfile`：定义容器映像和已安装的工具
- `init-firewall.sh`：建立网络安全规则

### 安全功能

该容器通过其防火墙配置实现多层安全方法：

- **精确的访问控制**：仅将出站连接限制为白名单域（npm 注册表、GitHub、Anthropic API 等）
- **默认拒绝策略**：阻止所有其他外部网络访问
- **启动验证**：在容器初始化时验证防火墙规则
- **隔离**：创建一个与你的主系统分离的安全开发环境

### 自定义选项

devcontainer 配置旨在适应你的需求：

- 根据你的工作流程添加或删除 VS Code 扩展
- 修改不同硬件环境的资源分配
- 调整网络访问权限
- 自定义 shell 配置和开发人员工具
