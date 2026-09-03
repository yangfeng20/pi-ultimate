# pi-ultimate 🚀

**把 Pi 从"能用"打磨成"好用、丝滑、能扛工作"的完全体教程。**

市面上的 Pi 教程大多停留在"怎么入门"：怎么装、怎么跑第一条命令。但装完之后呢？命令跑着跑着卡死了、grep 一把梭扫到 node_modules 卡了三分钟、想挂个定时任务却发现跨会话失效、终端里滚屏和输入框互相打架……

这些问题没有教程告诉你怎么解决。本仓库就是来补齐这一段的：**一套经过真实生产环境验证的 Pi 增强方案**，涵盖命令执行环境、TUI 交互、搜索工具、代理、定时任务工作流等。每个模块都来自真实的日常开发痛点，不是纸上谈兵。

## 🎯 你将获得什么

| 阶段 | 模块 | 能力 | 解决的痛点 |
|---|---|---|---|
| **基础层**（丝滑化） | M1 | 命令执行环境 | Windows 下 cmd 中文乱码、引号嵌套地狱 |
| | M2 | TUI 模式 | 滚屏与输入框打架、长输出难回看 |
| | M3 | 命令超时 | 命令卡死拖垮整个会话 |
| | M4 | 搜索工具链 | grep 扫 node_modules 卡三分钟、GBK 乱码 |
| | M5 | 命令别名 | 高频长命令重复输入 |
| | M6 | 代理设置 | 访问 GitHub / 搜索国外资源失败 |
| **增强层**（工作流） | M7 | pi-subagents | 想让子 agent 扛活但不会管 |
| | M8 | pi-schedule-status | 定时任务跑了没跑全靠猜 |
| | M9 | 工作流模板 | bug 巡检、值班排查全靠人肉盯 |

## 📦 一句话安装

不想 clone 仓库？把下面这段话**原样复制发给你的 Pi**：

```
请帮我安装 pi-ultimate 引导 skill，步骤：
1. 访问 https://raw.githubusercontent.com/yangfeng20/pi-ultimate/main/skills/pi-ultimate-setup/SKILL.md 获取内容
2. 将内容保存到 ~/.pi/agent/skills/pi-ultimate-setup/SKILL.md
3. 保存完成后，按照 skill 的指引对我的 Pi 环境进行检测与增强
```

<details>
<summary>🌐 raw 直连不稳？用 jsdelivr 镜像兜底</summary>

```
请帮我安装 pi-ultimate 引导 skill，步骤：
1. 访问 https://cdn.jsdelivr.net/gh/yangfeng20/pi-ultimate@main/skills/pi-ultimate-setup/SKILL.md 获取内容
2. 将内容保存到 ~/.pi/agent/skills/pi-ultimate-setup/SKILL.md
3. 保存完成后，按照 skill 的指引对我的 Pi 环境进行检测与增强
```

</details>

<details>
<summary>🪟 AI 无法直接访问 GitHub？手动安装兜底</summary>

1. 浏览器打开 <https://raw.githubusercontent.com/yangfeng20/pi-ultimate/main/skills/pi-ultimate-setup/SKILL.md>
2. 全选复制内容
3. 创建目录 `~/.pi/agent/skills/pi-ultimate-setup/`（Windows 为 `C:\Users\<你的用户名>\.pi\agent\skills\pi-ultimate-setup\`）
4. 新建 `SKILL.md` 并粘贴保存
5. 对 Pi 说：`按照 pi-ultimate-setup skill 的指引，对我的环境进行检测与增强`

</details>

## 🧭 模块导航

每个模块对应 `docs/` 一篇文档。**建议直接用"一句话安装"让 AI 引导你完成配置，文档作为深入理解与视频教程素材。**

### 基础层（丝滑化）

| 模块 | 文档 | 一句话说明 |
|---|---|---|
| M1 | [docs/m1-command-execution.md](docs/m1-command-execution.md) | Windows 命令执行环境（Git Bash / 编码 / 引号策略） |
| M2 | [docs/m2-tui-mode.md](docs/m2-tui-mode.md) | TUI 模式（fullscreen / sticky 输入框） |
| M3 | [docs/m3-timeout.md](docs/m3-timeout.md) | 命令超时规范（防卡死） |
| M4 | [docs/m4-grep.md](docs/m4-grep.md) | 搜索工具链（rg 安装 + grep 排除模式） |
| M5 | [docs/m5-alias.md](docs/m5-alias.md) | 命令别名 |
| M6 | [docs/m6-proxy.md](docs/m6-proxy.md) | 代理设置（不开全局，按需生效） |

### 增强层（工作流）

| 模块 | 文档 | 一句话说明 |
|---|---|---|
| M7 | [docs/m7-subagents.md](docs/m7-subagents.md) | pi-subagents 插件 + sessionOnly 定时任务 |
| M8 | [docs/m8-schedule-status.md](docs/m8-schedule-status.md) | pi-schedule-status 状态栏插件 |
| M9 | [docs/m9-workflow-template.md](docs/m9-workflow-template.md) | 通用定时巡检工作流模板（bug 源可换） |

## 💡 设计理念

- **AI 当装配工，你只当验收员**：所有配置由 AI 自动执行，你只提供缺失信息（如代理端口）
- **体检报告先行**：先全量检测给出 ✅/⚠️ 报告，再逐项询问是否开启，绝不静默改配置
- **模块化 + 可扩展**：每个模块独立，可单独启用/跳过；新模块按同一模式接入，老用户重跑引导即可补装
- **全平台覆盖**：Windows（Git Bash）与 macOS 双路径，Linux 用户同样适用

## 📄 License

MIT
