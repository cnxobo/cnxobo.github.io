---
title: Windows PowerShell 终端工具箱：9 个让你效率翻倍的 CLI 工具
tags:
  - Windows
  - PowerShell
  - CLI
  - scoop
categories:
  - - ops
date: 2026-03-16
---

Windows 的 PowerShell 功能强大但默认体验朴素，装上这 9 个工具后完全是另一个世界。本文以 Scoop 为包管理器，介绍文件浏览、内容搜索、目录跳转、版本管理等日常高频场景的现代 CLI 工具，所有配置均针对 PowerShell 编写。

<!-- more -->

## 前置准备

### 安装 Scoop

如果还没有 Scoop，在 PowerShell 中执行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

### 一键安装全部工具

```powershell
scoop install eza bat fd ripgrep fzf zoxide atuin starship mise
```

> **Nerd Font 提示**：`eza` 和 `starship` 依赖 Nerd Font 才能正确显示图标。推荐安装 JetBrainsMono Nerd Font：
>
> ```powershell
> scoop bucket add nerd-fonts
> scoop install nerd-fonts/JetBrainsMono-NF
> ```
>
> 安装后在 Windows Terminal 的设置中，将字体切换为 `JetBrainsMono Nerd Font`。

### PowerShell 配置文件

本文所有配置均写入 PowerShell 的 `$PROFILE` 文件。查看路径：

```powershell
echo $PROFILE
# 通常为: C:\Users\<用户名>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
```

如果文件不存在，先创建：

```powershell
if (!(Test-Path -Path $PROFILE)) { New-Item -ItemType File -Path $PROFILE -Force }
```

用编辑器打开：

```powershell
notepad $PROFILE
```

---

## 1. eza — 更好的 ls

`eza` 是 `ls` / `Get-ChildItem` 的现代替代品，支持颜色、图标、Git 状态和 tree 视图。

```powershell
scoop install eza
```

### 推荐配置

在 `$PROFILE` 中添加：

```powershell
function ll { eza -alh --icons --group-directories-first --git @args }
function la { eza -a --icons --group-directories-first @args }
function tree { eza --tree --icons --level=3 @args }
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `--icons` | 显示文件类型图标（需 Nerd Font） |
| `-l` | 长格式 |
| `-a` | 显示隐藏文件 |
| `--git` | 显示 Git 状态 |
| `--tree` | 树状视图 |
| `--level=N` | 树状视图深度 |
| `--group-directories-first` | 目录排在前面 |
| `--sort=modified` | 按修改时间排序 |

### 使用示例

```powershell
# 场景：快速查看项目结构
eza --tree --level=2 --icons

# 场景：查看目录下文件的 Git 状态
eza -alh --git --icons

# 场景：按修改时间排序，找最近改过的文件
eza -l --sort=modified --reverse

# 场景：只看目录
eza -D --icons

# 场景：带头部信息的详细列表
eza -lh --header --icons
```

---

## 2. bat — 带语法高亮的 cat

`bat` 为 `cat` / `Get-Content` 添加了语法高亮、行号和 Git diff 标记。

```powershell
scoop install bat
```

### 推荐配置

创建配置文件。先确定配置目录：

```powershell
# bat 在 Windows 上的配置路径
mkdir -Force "$env:APPDATA\bat"
```

创建 `%APPDATA%\bat\config`：

```
--theme="Catppuccin Mocha"
--style="numbers,changes,header,grid"
--italic-text=always
--map-syntax "*.conf:INI"
--map-syntax ".ignore:Git Ignore"
```

在 `$PROFILE` 中添加：

```powershell
function cat { bat --paging=never @args }
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `--theme=NAME` | 设置主题（`bat --list-themes` 查看） |
| `--style=COMPONENTS` | 显示组件：numbers, changes, header, grid |
| `-p` / `--plain` | 纯文本输出，不显示装饰 |
| `-l LANG` | 指定语言高亮 |
| `--paging=never` | 不使用分页器 |
| `-r N:M` | 只显示第 N 到 M 行 |
| `--diff` | 只显示 Git 修改的行 |

### 使用示例

```powershell
# 场景：查看配置文件，带行号和语法高亮
bat nginx.conf

# 场景：查看文件的特定行范围
bat -r 20:40 main.go

# 场景：对比文件修改（只看 Git diff 部分）
bat --diff src/app.ts

# 场景：管道输入并指定语言高亮
Invoke-RestMethod https://api.example.com/data | bat -l json

# 场景：列出所有支持的主题
bat --list-themes | Select-Object -First 20
```

---

## 3. fd — 更简单的 find

`fd` 是 `find` / `Get-ChildItem -Recurse` 的替代品，语法更直观，默认忽略 `.gitignore` 中的文件，速度更快。

```powershell
scoop install fd
```

### 推荐配置

创建忽略文件 `%APPDATA%\fd\ignore`：

```powershell
mkdir -Force "$env:APPDATA\fd"
```

```
node_modules
.git
target
dist
build
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `-e EXT` | 按扩展名过滤 |
| `-t f` / `-t d` | 只找文件 / 只找目录 |
| `-H` | 搜索隐藏文件 |
| `-I` | 不忽略 `.gitignore` |
| `-d N` | 限制搜索深度 |
| `-x CMD` | 对每个结果执行命令 |
| `-X CMD` | 对所有结果批量执行命令 |
| `-E PATTERN` | 排除匹配的路径 |

### 使用示例

```powershell
# 场景：在项目中查找所有 Go 文件
fd -e go

# 场景：查找名字包含 test 的文件
fd test

# 场景：查找最近 1 天内修改过的文件
fd --changed-within 1d

# 场景：查找所有 Dockerfile 并显示内容
fd Dockerfile -x bat {}

# 场景：查找大于 10MB 的文件
fd -t f -S +10m
```

---

## 4. ripgrep (rg) — 极速文本搜索

`ripgrep` 是 `grep` / `Select-String` 的替代品，速度极快，默认递归搜索且遵守 `.gitignore`。

```powershell
scoop install ripgrep
```

### 推荐配置

创建配置文件并设置环境变量：

```powershell
mkdir -Force "$env:USERPROFILE\.config\ripgrep"
```

创建 `%USERPROFILE%\.config\ripgrep\config`：

```
--smart-case
--hidden
--glob=!.git
--max-columns=200
--max-columns-preview
```

在 `$PROFILE` 中添加：

```powershell
$env:RIPGREP_CONFIG_PATH = "$env:USERPROFILE\.config\ripgrep\config"
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `-i` | 忽略大小写 |
| `-S` / `--smart-case` | 智能大小写（有大写则精确匹配） |
| `-w` | 全词匹配 |
| `-t TYPE` | 指定文件类型（如 `-t py`） |
| `-g GLOB` | 用 glob 过滤文件 |
| `-l` | 只显示匹配的文件名 |
| `-c` | 显示每个文件的匹配数 |
| `-C N` | 显示上下文行数 |
| `--json` | 输出 JSON 格式 |

### 使用示例

```powershell
# 场景：在项目中搜索 TODO 注释
rg TODO

# 场景：搜索某个函数定义
rg "func\s+handleRequest"

# 场景：在指定类型文件中搜索
rg "import.*React" -t tsx -t jsx

# 场景：统计每个文件的匹配数
rg -c "fmt.Println" -t go

# 场景：搜索并替换（配合 PowerShell）
rg -l "oldFunc" | ForEach-Object { (Get-Content $_) -replace 'oldFunc', 'newFunc' | Set-Content $_ }
```

---

## 5. fzf — 模糊搜索粘合剂

`fzf` 是一个通用的模糊查找器，可以和几乎任何命令组合，充当交互式过滤器。在 PowerShell 中推荐搭配 `PSFzf` 模块使用。

```powershell
scoop install fzf
Install-Module -Name PSFzf -Scope CurrentUser
```

### 推荐配置

在 `$PROFILE` 中添加：

```powershell
# fzf 默认选项
$env:FZF_DEFAULT_OPTS = @"
--height=60%
--layout=reverse
--border=rounded
--prompt='> '
--pointer='▶'
--marker='✓'
--preview-window=right:60%:wrap
"@

# 使用 fd 作为默认搜索命令
$env:FZF_DEFAULT_COMMAND = "fd --type f --hidden --exclude .git"

# Ctrl+T 用 fd 搜索文件
$env:FZF_CTRL_T_COMMAND = $env:FZF_DEFAULT_COMMAND
$env:FZF_CTRL_T_OPTS = "--preview 'bat --color=always --style=numbers --line-range=:500 {}'"

# Alt+C 用 fd 搜索目录
$env:FZF_ALT_C_COMMAND = "fd --type d --hidden --exclude .git"
$env:FZF_ALT_C_OPTS = "--preview 'eza --tree --level=2 --icons {}'"

# 启用 PSFzf 模块
Import-Module PSFzf

# 绑定快捷键
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `--height=N%` | 占用终端高度比例 |
| `--layout=reverse` | 从顶部开始显示 |
| `--preview=CMD` | 右侧预览窗口 |
| `--multi` / `-m` | 允许多选（Tab 选中） |
| `--query=STR` | 初始搜索关键词 |
| `--filter=STR` | 非交互式过滤 |
| `--bind=KEY:ACTION` | 自定义键绑定 |

### 使用示例

```powershell
# 场景：交互式选择文件并用 VS Code 打开
fd --type f | fzf | ForEach-Object { code $_ }

# 场景：搜索并切换 Git 分支
git branch -a | fzf | ForEach-Object { git checkout $_.Trim() }

# 场景：交互式杀进程
Get-Process | ForEach-Object { "$($_.Id)`t$($_.ProcessName)" } | fzf | ForEach-Object { Stop-Process -Id ($_ -split "`t")[0] }

# 场景：Ctrl+R 交互式搜索历史命令（PSFzf 自动接管）
# 直接按 Ctrl+R 即可

# 场景：多选文件并删除
fd -t f | fzf -m | ForEach-Object { Remove-Item $_ }
```

---

## 6. zoxide — 智能 cd

`zoxide` 记录你访问过的目录，让你用部分路径名就能跳转到目标目录。

```powershell
scoop install zoxide
```

### 推荐配置

在 `$PROFILE` 中添加：

```powershell
Invoke-Expression (& { (zoxide init powershell | Out-String) })
```

### 常用命令

| 命令 | 说明 |
|------|------|
| `z keyword` | 跳转到匹配的最高权重目录 |
| `z keyword1 keyword2` | 多关键词缩小范围 |
| `zi` | 交互式选择（需要 fzf） |
| `zoxide query keyword` | 查看匹配结果但不跳转 |
| `zoxide edit` | 编辑数据库 |

### 使用示例

```powershell
# 场景：跳转到之前访问过的项目目录
z myproject

# 场景：多个同名目录时用多关键词区分
z repo frontend   # 匹配 C:\code\repo\frontend

# 场景：交互式选择目录（fzf 风格）
zi

# 场景：查看某个关键词匹配了哪些目录
zoxide query blog --list
```

---

## 7. atuin — Shell 历史数据库

`atuin` 将 shell 历史存储在 SQLite 数据库中，支持全文搜索、跨会话同步和统计分析。

```powershell
scoop install atuin
```

### 推荐配置

配置文件路径：`%USERPROFILE%\.config\atuin\config.toml`

```powershell
mkdir -Force "$env:USERPROFILE\.config\atuin"
```

```toml
[settings]
# 搜索模式：prefix | fulltext | fuzzy | skim
search_mode = "fuzzy"
# 过滤模式：global | host | session | directory
filter_mode = "global"
# 按时间倒序
style = "compact"
# 内联显示高度
inline_height = 20
# 不记录失败的命令
exit_filter = "success"
# 同步（可选，不需要可注释）
# sync_address = "https://api.atuin.sh"
# sync_frequency = "5m"
```

在 `$PROFILE` 中添加：

```powershell
Invoke-Expression (& { (atuin init powershell | Out-String) })
```

> **注意**：atuin 的 PowerShell 支持相对较新，如遇到兼容性问题，可先用 `PSFzf` 的 `Ctrl+R` 作为历史搜索的替代方案。

### 常用选项

| 命令 / 快捷键 | 说明 |
|------|------|
| `Ctrl+R` | 交互式搜索历史 |
| `atuin search keyword` | 搜索历史命令 |
| `atuin stats` | 显示命令使用统计 |
| `atuin history list` | 列出历史记录 |
| `atuin import auto` | 从现有 shell 历史导入 |

### 使用示例

```powershell
# 场景：搜索之前执行过的 docker 命令
atuin search docker

# 场景：查看命令使用统计（最常用的命令）
atuin stats

# 场景：搜索特定目录下执行过的命令
atuin search --cwd C:\project docker

# 场景：首次安装后导入现有历史
atuin import auto
```

---

## 8. starship — 美化命令提示符

`starship` 是一个极快的、可定制的跨 shell 提示符，自动显示 Git 状态、语言版本、执行时间等信息。

```powershell
scoop install starship
```

### 推荐配置

配置文件路径：`%USERPROFILE%\.config\starship.toml`

```toml
# 整体格式
format = """
$directory\
$git_branch\
$git_status\
$nodejs\
$python\
$golang\
$java\
$rust\
$docker_context\
$cmd_duration\
$line_break\
$character"""

[character]
success_symbol = "[❯](bold green)"
error_symbol = "[❯](bold red)"

[directory]
truncation_length = 3
truncate_to_repo = true
style = "bold cyan"

[git_branch]
format = "[$symbol$branch]($style) "
symbol = " "

[git_status]
format = '([\[$all_status$ahead_behind\]]($style) )'

[cmd_duration]
min_time = 2_000
format = "[$duration]($style) "

[nodejs]
format = "[$symbol($version)]($style) "
symbol = " "

[python]
format = "[$symbol($version)]($style) "
symbol = " "

[golang]
format = "[$symbol($version)]($style) "
symbol = " "

[java]
format = "[$symbol($version)]($style) "
symbol = " "
```

在 `$PROFILE` 中添加：

```powershell
Invoke-Expression (&starship init powershell)
```

### 常用配置项

| 模块 | 说明 |
|------|------|
| `[directory]` | 当前路径显示 |
| `[git_branch]` | Git 分支名 |
| `[git_status]` | Git 状态（修改、暂存等） |
| `[cmd_duration]` | 命令执行时间 |
| `[nodejs]` / `[python]` / `[golang]` | 语言版本 |
| `[docker_context]` | Docker 上下文 |
| `[character]` | 提示符字符 |

### 使用示例

```powershell
# 场景：使用预设快速开始
starship preset gruvbox-rainbow -o "$env:USERPROFILE\.config\starship.toml"

# 场景：查看所有可用模块
starship module --list

# 场景：调试提示符（查看各模块耗时）
starship timings

# 场景：解释当前提示符各段含义
starship explain
```

---

## 9. mise — 开发工具版本管理

`mise`（前身为 rtx）管理 Node.js、Python、Go、Java 等开发工具的多版本，兼容 `.tool-versions` 和 `.node-version` 等格式。

```powershell
scoop install mise
```

### 推荐配置

在 `$PROFILE` 中添加：

```powershell
Invoke-Expression (& { (mise activate powershell | Out-String) })
```

全局配置文件：`%USERPROFILE%\.config\mise\config.toml`

```toml
[tools]
node = "lts"
python = "3.12"
golang = "1.22"
java = "21"

[settings]
experimental = true
```

### 常用命令

| 命令 | 说明 |
|------|------|
| `mise use node@20` | 在当前目录设置 Node.js 版本 |
| `mise use -g node@lts` | 设置全局默认版本 |
| `mise install` | 安装当前配置所需的所有工具 |
| `mise ls` | 列出已安装的工具和版本 |
| `mise ls-remote node` | 查看可安装的远程版本 |
| `mise current` | 显示当前激活的版本 |
| `mise prune` | 清理未使用的版本 |

### 使用示例

```powershell
# 场景：新项目需要特定 Node.js 版本
Set-Location ~/project
mise use node@20.11.0
# 自动生成 .mise.toml，进入目录自动切换

# 场景：查看当前目录激活的所有工具版本
mise current

# 场景：安装多个 Python 版本并切换
mise use python@3.11 python@3.12
mise use python@3.12  # 设为默认

# 场景：用 mise 运行特定版本的工具
mise exec node@18 -- node -e "console.log(process.version)"

# 场景：清理磁盘空间
mise prune
```

---

## 组合技

单独使用这些工具已经很好，组合起来才是真正的效率飞跃。

### 1. fd + fzf + bat — 模糊搜索文件并预览

```powershell
# 用 fd 列出文件，fzf 交互选择，bat 实时预览内容，VS Code 打开
fd --type f | fzf --preview "bat --color=always --style=numbers {}" | ForEach-Object { code $_ }
```

用途：在大型项目中快速定位并打开文件。

### 2. rg + fzf + bat — 交互式代码搜索

```powershell
# 搜索代码内容，交互式选择，预览上下文
rg --line-number --color=always "" |
  fzf --ansi --delimiter=":" `
    --preview "bat --color=always --highlight-line {2} {1}" `
    --preview-window="right:60%:+{2}-10"
```

用途：在项目中搜索代码片段，预览上下文后定位到对应行。

### 3. fd + rg — 精确范围搜索

```powershell
# 只在 Go 文件中搜索 error handling 模式
fd -e go | ForEach-Object { rg "if err != nil" $_ }

# 在最近修改的文件中搜索
fd --changed-within 2d | ForEach-Object { rg "TODO" $_ }
```

用途：先缩小文件范围，再精确搜索内容。

### 4. zoxide + fzf — 交互式目录跳转

```powershell
# zi 命令 = zoxide 交互模式（需要 fzf）
zi
```

用途：当你记不清完整路径时，交互式浏览并跳转到历史目录。

### 5. eza + fzf + bat — 浏览目录并预览文件

```powershell
# 用 eza 列出文件，fzf 选择，bat 预览
eza --icons -1 | fzf --preview "bat --color=always {} 2>`$null; if (`$LASTEXITCODE -ne 0) { eza --tree --level=2 --icons {} }"
```

用途：在当前目录中快速浏览，文件显示内容预览，目录显示结构预览。

### 6. mise + starship — 版本信息自动显示

只要在项目目录中配置了 `.mise.toml` 或 `.tool-versions`，`mise` 会自动激活对应版本，`starship` 会在提示符中实时显示当前语言版本。无需额外配置，开箱即用。

```
C:\code\my-project (main)  v20.11.0  v3.12.0 ❯
```

---

## 最佳实践

### 1. Nerd Font 是基础

`eza` 的图标和 `starship` 的符号都依赖 Nerd Font。推荐 **JetBrainsMono Nerd Font**，等宽且图标覆盖完整。安装后务必在 **Windows Terminal** 的设置 → 配置文件 → 外观中切换字体。

### 2. 使用 Windows Terminal

强烈推荐使用 **Windows Terminal** 而不是旧版 `cmd.exe` 窗口。Windows Terminal 对 Unicode、Nerd Font、ANSI 颜色的支持远好于传统终端。从 Microsoft Store 安装即可。

### 3. PowerShell 初始化顺序

`$PROFILE` 中各工具的初始化顺序会影响功能。推荐顺序：

```powershell
# 1. 版本管理（最先，确保 PATH 正确）
Invoke-Expression (& { (mise activate powershell | Out-String) })

# 2. 目录跳转
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# 3. 提示符（依赖 mise 的环境变量）
Invoke-Expression (&starship init powershell)

# 4. 历史管理（覆盖默认 Ctrl+R）
Invoke-Expression (& { (atuin init powershell | Out-String) })

# 5. 模糊搜索（最后，绑定快捷键）
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'
```

### 4. 渐进式采纳

不要一口气替换所有命令。PowerShell 中用 `function` 定义别名更灵活，建议分阶段采纳：

```powershell
# 第一阶段：新命令名，不影响肌肉记忆
function ll { eza -alh --icons --git @args }
function rgl { rg -l @args }

# 第二阶段：确认顺手后覆盖内置命令
function cat { bat --paging=never @args }
Set-Alias -Name grep -Value rg
```

> **提示**：PowerShell 的 `Set-Alias` 不支持带参数的别名，需要用 `function` 代替。

### 5. 配置文件纳入版本管理

将 dotfiles 用 Git 管理，方便在多台机器间同步：

```
# 需要管理的配置文件
$PROFILE                                    # PowerShell 配置
%USERPROFILE%\.config\starship.toml         # Starship 提示符
%USERPROFILE%\.config\atuin\config.toml     # Atuin 历史
%APPDATA%\bat\config                        # Bat 主题样式
%USERPROFILE%\.config\mise\config.toml      # Mise 版本管理
```

### 6. 性能注意事项

- `starship` 的模块按需加载，不用的模块设为 `disabled = true` 可加速提示符渲染
- `eza --git` 在超大仓库中可能较慢，可按需使用
- `fzf` 的 `--preview` 对大文件可能卡顿，建议加 `--line-range=:500` 限制预览范围
- PowerShell 启动速度受 `$PROFILE` 影响，可用 `Measure-Command { pwsh -NoProfile -Command 1 }` 对比基线
- 如果启动明显变慢，考虑延迟加载不常用的模块

---

## 完整 $PROFILE 配置

以下是可直接复制的完整配置块：

```powershell
# ==========================================
# Windows PowerShell CLI 工具箱 - $PROFILE 配置
# ==========================================

# --- mise (版本管理，最先加载) ---
Invoke-Expression (& { (mise activate powershell | Out-String) })

# --- zoxide (智能 cd) ---
Invoke-Expression (& { (zoxide init powershell | Out-String) })

# --- starship (提示符) ---
Invoke-Expression (&starship init powershell)

# --- atuin (历史管理) ---
Invoke-Expression (& { (atuin init powershell | Out-String) })

# --- PSFzf (模糊搜索) ---
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'

# --- ripgrep ---
$env:RIPGREP_CONFIG_PATH = "$env:USERPROFILE\.config\ripgrep\config"

# --- fzf 选项 ---
$env:FZF_DEFAULT_OPTS = @"
--height=60%
--layout=reverse
--border=rounded
--prompt='> '
--pointer='▶'
--marker='✓'
--preview-window=right:60%:wrap
"@
$env:FZF_DEFAULT_COMMAND = "fd --type f --hidden --exclude .git"
$env:FZF_CTRL_T_COMMAND = $env:FZF_DEFAULT_COMMAND
$env:FZF_CTRL_T_OPTS = "--preview 'bat --color=always --style=numbers --line-range=:500 {}'"
$env:FZF_ALT_C_COMMAND = "fd --type d --hidden --exclude .git"
$env:FZF_ALT_C_OPTS = "--preview 'eza --tree --level=2 --icons {}'"

# --- 别名 ---
function ll { eza -alh --icons --group-directories-first --git @args }
function la { eza -a --icons --group-directories-first @args }
function tree { eza --tree --icons --level=3 @args }
function cat { bat --paging=never @args }
Set-Alias -Name grep -Value rg
```
