---
name: pi-ultimate-setup
description: Pi 完全体引导。对用户的 Pi 环境做全量体检，报告缺失项，逐项引导增强（命令环境、TUI、超时、搜索工具、别名、代理、子 agent 插件、定时任务状态栏、工作流模板）。当用户说"帮我安装 pi-ultimate"、"搭建 pi 完全体"、"检测/增强我的 pi 环境"、"看看我的 pi 还缺什么"时使用。
---

# Pi Ultimate Setup — Pi 完全体引导 skill

把用户的 Pi 从"能用"打磨成"好用、丝滑、能扛工作"。**AI 当装配工，用户只当验收员**。

## 0. 铁律（最高优先级）

1. **先体检，再询问，绝不静默改配置**：每一项改动都要先说清"痛点 + 将要做什么"，得到用户同意后才动手。
2. **AI 自己动手**：凡 AI 能自动完成的配置（写 settings.json、追加 AGENTS.md、装插件、装 brew/scoop 包）一律自动执行；只有 AI 做不了的事（登录、输入 token/密码、点授权）才教用户手动。
3. **缺信息才问，问一次**：需要用户提供的信息（如代理端口）一次性收集完，不要反复打断。
4. **脱敏**：本 skill 是公开模板，任何位置不得写入用户个人或公司敏感信息（账号、域名、token、内部系统名）。示例一律用占位符（`<your>`、`example.com`）。
5. **失败可回滚**：每次改配置前先备份（`cp x x.bak` 或记下原内容），保证随时可还原。
6. **模块化**：每个模块独立检测、独立询问、独立开关。用户拒绝的模块记为"未启用"，报告里列出，绝不纠缠。

## 1. 启动引导

当用户触发本 skill 时：

1. 欢迎 + 说明流程：先做一次全量体检（约 1 分钟），然后逐项询问是否增强。
2. 收集一次性信息（缺的才问）：
   - 操作系统（自动探测，无需问）
   - 代理端口（如用户提到网络问题 / 有代理软件时才问；否则跳过该项）
   - 常用命令别名需求（可选，不问则用默认推荐集）
3. 开始体检。

## 2. 环境探测

自动执行（不要逐个询问）：

```
# OS
uname -s        # Darwin / Linux / MINGW*（Windows Git Bash）
echo $OSTYPE

# Pi 版本
pi --version 2>/dev/null || npx pi --version 2>/dev/null

# 包管理器
which brew scoop winget choco 2>/dev/null

# 关键工具
which rg git bash 2>/dev/null

# Pi 配置目录
echo ~/.pi
ls ~/.pi/agent/settings.json 2>/dev/null
ls ~/.pi/agent/AGENTS.md 2>/dev/null

# 已装插件 / skills
ls ~/.pi/agent/extensions/ 2>/dev/null
ls ~/.pi/agent/skills/ 2>/dev/null
```

记录到体检表：`os`、`piVersion`、`hasRg`、`hasBrew/hasScoop`、`settingsExists`、`agentsExists`、`extensions`、`skills`。

## 3. 体检清单（模块表）

对每个模块输出一行：`✅ 已就绪 / ⚠️ 缺失（痛点：xxx）/ ⏭ 不适用（如 macOS 跳过 Windows 项）`。

| 模块 | 检测条件 | 就绪判定 |
|---|---|---|
| M1 命令执行环境 | Windows 才检测 | Git Bash 可用 且 PATH 中 `Git\bin` 排在 `System32` 前；AGENTS.md 含编码/引号规范 |
| M2 TUI 模式 | 全部平台 | settings.json 中 `tuiMode` 为 `fullscreen` |
| M3 命令超时 | 全部平台 | AGENTS.md 含 timeout 规范（grep ≤180s / 其他 ≤300s） |
| M4 搜索工具链 | 全部平台 | rg 已安装；AGENTS.md 含排除产物目录 + rg 优先规则 |
| M5 命令别名 | 全部平台 | shell 配置文件（.bashrc / .zshrc）含常用别名 |
| M6 代理设置 | 全部平台 | 已配置按需代理（HTTPS_PROXY 等）或 AGENTS.md 有代理说明 |
| M7 pi-subagents | 全部平台 | 插件已装且版本 ≥0.62（sessionOnly 能力） |
| M8 pi-schedule-status | 全部平台 | 官方扩展库已安装该插件 |
| M9 工作流模板 | 全部平台 | 已有自定义巡检/值班类 skill 或 AGENTS.md 有相关流程 |

## 4. 输出体检报告

按模块逐行列出。示例：

```
✅ M1 命令执行环境   已就绪（Git Bash 正常，编码/引号规范已注入）
⚠️ M2 TUI 模式       未开启（痛点：长输出滚屏与输入框打架，回看困难）
⚠️ M4 搜索工具链     rg 未安装（痛点：grep 扫 node_modules 会卡到怀疑人生）
⏭ M6 代理设置       跳过（本次未提供代理信息）
```

然后进入逐项处理：**只处理 ⚠️ 项，✅/⏭ 跳过**。

## 5. 逐模块增强剧本

每个模块的处理步骤统一为：**① 简述痛点 → ② 询问是否开启 → ③ 同意则自动配置并验证 → ④ 记录结果**。

### M1 命令执行环境（仅 Windows）

- 痛点：cmd 中文乱码、命令卡在 PowerShell 编码、引号嵌套三层就爆炸。
- 检测细节：
  - `which bash` 是否指向 `Git\bin\bash.exe`（而非 WSL）
  - PATH 顺序：`echo $PATH | tr ':' '\n' | grep -nE 'Git/bin|Git/usr/bin|System32'`
- 增强动作：
  - 如 bash 指向 WSL：提示安装 Git for Windows（`winget install Git.Git`），并说明把 `Git\bin`、`Git\usr\bin` 加到系统 PATH 且**排在 System32 之前**。
  - 将"Windows 命令兼容规范"追加到 AGENTS.md（见附录 A），先展示将追加内容 → 确认 → 写入。
- 验证：重跑 `which bash`；`echo "中文"` 输出正常。

### M2 TUI 模式

- 痛点：默认非 fullscreen 模式下长输出与输入框互相挤占，回看历史输出困难。
- 增强动作：把 `"tuiMode": "fullscreen"` 写入 `~/.pi/agent/settings.json`（若不存在则创建；先备份）。
- 说明：fullscreen 提供独立可滚动模型输出区 + 底部固定输入框（sticky），更贴近"完整 IDE 终端"体验。部分用户不习惯可随时改回。
- 验证：读取 settings.json 确认写入成功；提示用户下次重启 Pi 生效。

### M3 命令超时规范

- 痛点：没有超时上限，一条卡死的命令能拖垮整个会话；超时设得过大等于没有。
- 增强动作：将"命令超时规范"（见附录 A）注入 AGENTS.md。
- 验证：确认 AGENTS.md 含 `timeout` 关键字。

### M4 搜索工具链（rg + grep 排除模式）

- 痛点：grep 不带排除会扫 node_modules/target/dist 卡到怀疑人生；Windows 下 grep 还常遇到编码问题。
- 增强动作：
  1. 安装 rg：
     - macOS：`brew install ripgrep`
     - Windows：`winget install BurntSushi.ripgrep.MSVC` 或 `scoop install ripgrep`
     - Linux：`apt install ripgrep` / `yum install ripgrep`
  2. 若 AI 无法安装（如无权限），给出命令让用户手动执行，然后继续。
  3. 将"搜索规范"（grep 一律用 rg，或 grep 必须排除产物目录）注入 AGENTS.md（见附录 A）。
- 验证：`which rg`；`rg --version`；在示例目录跑一次 `rg "TODO"` 确认生效。

### M5 命令别名

- 痛点：高频长命令（`git status`、`git log --oneline`、`ls -la` 等）重复输入累。
- 增强动作：
  - 探测 shell：`echo $SHELL`；Windows Git Bash 为 `.bashrc`，macOS 为 `.zshrc`。
  - 将推荐别名集（见附录 B）追加到对应文件（先备份 + 去重）。
  - 若用户有自己惯用的别名，让用户提供，追加时保留。
- 验证：`source` 后 `type <别名>` 能解析。

### M6 代理设置

- 痛点：国内访问 GitHub / 搜索国外资源失败，但不想开全局代理。
- 增强动作：
  - 询问代理地址与端口（如 `127.0.0.1:7890`），一次问清。
  - 按平台写入按需代理：
    - 方案 A（推荐，按命令生效）：在 AGENTS.md 写明"访问国外资源时使用 `https_proxy=http://<host>:<port>`"，AI 在需要时自动带上。
    - 方案 B（环境变量，可选）：写 shell profile，仅对子进程生效。
  - 若用户无代理：跳过并在报告中注明，不阻塞其他模块。
- 验证：`curl -x http://<host>:<port> -I https://github.com` 返回 200。

### M7 pi-subagents（sessionOnly 能力）

- 痛点：定时任务被任意会话抢走执行，或想隔离"仅当前会话执行"的任务。
- 增强动作：
  - 检查 `~/.pi/agent/extensions/` 是否有 pi-subagents，版本 ≥0.62。
  - 未装/版本旧：引导安装/升级到 ≥0.62（支持 `sessionOnly` 参数）。
  - 在 AGENTS.md 注入"定时任务创建规范"（见附录 A）：需要仅当前会话执行时用 sessionOnly。
- 验证：`pi --version`；读取扩展清单确认版本。

### M8 pi-schedule-status 状态栏插件

- 痛点：定时任务跑了没跑、下次什么时候跑，全靠猜，肉眼不可见。
- 增强动作：
  - 未安装：从 pi 官方扩展库安装 `pi-schedule-status`（`pi install pi-schedule-status` 或按官方指引）。
  - 装好后提醒用户重启 Pi 使状态栏生效，并说明它会显示：任务名、ON/OFF、下次执行时间、执行中状态。
- 验证：读取扩展清单确认已安装；提示重启。

### M9 工作流模板（通用定时巡检）

- 痛点：bug 巡检、值班排查这种重复劳动，全靠人肉盯着群 / bug 源，费神易漏。
- 增强动作：**不直接给死板模板**，而是引导用户抽象自己的场景：
  1. 问清三个要素：
     - **任务源**：bug / 工单从哪里来（TAPD、Jira、飞书项目、邮件、群消息……任意）
     - **触发方式**：定时轮询（推荐）还是事件推送
     - **处理闭环**：排查 → 修复 → 通知谁 → 状态更新
  2. 根据回答，给出一段**可落地的 skill 骨架**（通用版，占位符替换），引导用户确认后由 AI 落盘到 `~/.pi/agent/skills/<name>/SKILL.md`。
  3. 注入配套的"定时巡检规范"到 AGENTS.md（sessionOnly、静默无新单、子 agent 隔离等）。
- 通用模板骨架见附录 C。

## 6. 结束报告

全部模块处理完后，输出总结：

- ✅ 已启用模块清单
- ⏭ 未启用模块清单（含原因：用户拒绝 / 缺信息 / 不适用）
- 需要用户**手动/重启/登录**才能生效的项（如重启 Pi、安装系统包）
- 一句"下次重跑本 skill 可检测增量并补装新模块"

## 7. 升级与兼容

- 本仓库会持续新增模块。用户重跑引导时：已就绪模块自动跳过，只处理新增/缺失项。
- 新增模块遵循同一模式：检测条件 + 痛点 + 增强动作 + 验证，接入体检表即可。
- 向后兼容：不删除/不回退用户已有配置，只做增量追加；所有写入前先备份。

---

## 附录 A：AGENTS.md 规范片段（按模块注入）

> 注入时只追加对应段落，不要覆盖用户已有内容。先展示给用户确认。

### A.1 命令执行（Windows）

```
## Windows 命令执行兼容（强制）
- 所有命令优先走 Git Bash / PowerShell 通道，不要在 cmd 裸跑中文相关命令。
- 读 UTF-8 中文文件必须显式指定编码（PowerShell 用 -Encoding UTF8），或用 Git Bash cat。
- 引号策略：优先两层引号（外层双引号 + 内层单引号）；真三层嵌套用临时脚本文件。
```

### A.2 命令超时

```
## 命令超时规范（强制）
- bash 工具调用必须显式设置 timeout。
- grep 搜索类命令 ≤ 180s；其他命令 ≤ 300s；禁止省略、禁止超大值。
- 预计超时必须换替代方案（rg / 缩小目录 / 分批），不得硬等。
```

### A.3 搜索工具链

```
## 代码搜索规范（强制）
- grep 一律用 rg（自动跳过 node_modules/dist/target/.git）。
- 必须用 grep 时，必须加 --exclude-dir=node_modules --exclude-dir=target --exclude-dir=dist --exclude-dir=.git。
```

### A.4 定时任务

```
## 定时任务规范（强制）
- 创建定时任务时，默认确认是否"仅当前会话执行"（sessionOnly）。
- 巡检类任务：无新单静默，有新单才汇报。
- 子 agent 隔离：不同任务/单子放独立子 agent，主会话只做汇总。
```

## 附录 B：推荐别名集

```
# 基础
alias ls='ls -la'
alias ll='ls -la'
alias gs='git status'
alias gl='git log --oneline -10'
alias gp='git pull'
alias gpo='git push origin HEAD'
alias ..='cd ..'
alias ...='cd ../..'
```

## 附录 C：通用定时巡检 skill 骨架

```markdown
---
name: <your-workflow-name>
description: 定时巡检 <任务源>，发现 <事件> 后自动 <处理动作>，并通知 <对象>。
---

# <名称>

## 触发
用户说"开始 <名称> 巡检"时启动；启动后按 <间隔> 定时轮询。

## 任务源
- 来源：<TAPD / Jira / 飞书项目 / 群消息 / 邮件……>
- 拉取方式：<接口 / 查询 / 消息流>
- 新单判定：<状态变化 / 新增评论 / 重新打开……>

## 处理闭环
1. 拉取 → 2. 去重（按单号 + 状态感知）→ 3. 分配子 agent 排查
4. 判定（真 bug / 非代码问题）→ 5. 修复 + 验证 → 6. 通知 + 更新状态
- 非代码问题：不修代码，只加评论 + 通知。

## 环境注意
- 涉及线上问题时：使用生产最新基线代码分析，别用本地旧分支。
- 多环境（如不同云/地域）注意区分。

## 边界
- 仅当前会话执行（sessionOnly）。
- 无新单静默；有敏感操作（改库、发布）前必须征求用户确认。
```
