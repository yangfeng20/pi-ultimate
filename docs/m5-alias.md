# M5 命令别名

> 目标：高频长命令一键敲，少打字、少错。

## 痛点

- `git status`、`git log --oneline`、`ls -la` 这些天天敲，重复输入浪费生命。
- 长命令容易敲错，还难记住。

## 推荐别名集

```bash
# 基础
alias ls='ls -la'
alias ll='ls -la'
alias ..='cd ..'
alias ...='cd ../..'

# git
alias gs='git status'
alias gl='git log --oneline -10'
alias gd='git diff'
alias gp='git pull'
alias gpo='git push origin HEAD'
alias gcm='git commit -m'

# 常用
alias h='history'
alias untar='tar -xzf'
```

## 写入位置

按 shell 决定：

| shell | 配置文件 |
|---|---|
| Git Bash（Windows） | `~/.bashrc`（或 `~/.bash_profile`） |
| macOS / Linux | `~/.zshrc`（zsh）或 `~/.bashrc`（bash） |

AI 引导时：追加前先备份 + 去重（避免重复追加），追加后 `source` 生效。

## 自定义

告诉 AI 你惯用的别名，追加时保留你的习惯；不喜欢默认集的直接删掉对应行即可。

## 验证

```bash
source ~/.bashrc
type gs        # 应输出 gs 是 `git status` 的别名
```
