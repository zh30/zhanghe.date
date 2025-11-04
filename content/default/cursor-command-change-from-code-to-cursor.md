---
title: Cursor PATH 命令从 code 改为 cursor
date: 2025-02-08
---

要将 Mac 上 Cursor IDE 的 "code" CLI 快捷指令改回 "cursor" 默认指令，您可以按照以下步骤操作：

1.  **删除 "code" 符号链接**:
    - 打开终端应用程序。
    - 输入以下命令并按回车键：`rm /usr/local/bin/code`。这将删除 "code" 快捷指令的符号链接。

2.  **安装 "cursor" 命令**:
    - 在 Cursor IDE 中，按下 `CMD + Shift + P` 打开命令面板。
    - 输入 "Shell Command: Install 'cursor' command" 并选择该选项。这将安装 "cursor" 命令，允许您在终端中使用 `cursor` 命令打开 Cursor IDE。

或者，您也可以手动创建一个函数或别名来实现：

- 打开您的 `.zshrc` 或 `.bashrc` 文件。
- 添加以下函数（用于 zsh）或别名（用于 bash）：

  **Zsh:**

  ```zsh
  function cursor {
    open -a "/Applications/Cursor.app" "$@"
  }
  ```

  **Bash:**

  ```bash
  alias cursor='open -a "/Applications/Cursor.app"'
  ```

- 保存文件并运行 `source ~/.zshrc` 或 `source ~/.bashrc` 以应用更改。

通过这些步骤，您应该能够使用 `cursor` 命令从终端打开 Cursor IDE。
