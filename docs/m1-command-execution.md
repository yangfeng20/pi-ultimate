# M1 命令执行环境（Windows）

> 目标：让 Pi 在 Windows 上跑命令不踩"中文乱码 / 编码错乱 / 引号爆炸"三大坑。

## 痛点

- cmd.exe 中文字符串被硬编码为 GBK（代码页 936）解码，`chcp 65001` 实测无效，中文输出必乱码。
- PowerShell（opencode/pwsh 通道）不加载用户 profile，`[Console]::OutputEncoding` 设置不生效，读 UTF-8 中文文件会按 GBK 解码成乱码。
- 引号嵌套到三层以上时用 `\"` 硬转义，cmd 不认这个转义，命令必然被截断执行失败。

## 解法：Git Bash 通道 + 编码规范 + 引号策略

### 1. 确保 bash 指向 Git Bash 而非 WSL

```bash
which bash
# 期望输出：D:\Program Files\Git\bin\bash.exe（或你机器上的 Git 安装路径）
# 若指向 WSL 或 System32\bash.exe，需要把 Git\bin 加进系统 PATH 并排在 System32 之前
```

配置方法（Windows）：

1. 安装 [Git for Windows](https://git-scm.com/download/win)（`winget install Git.Git`）
2. 系统 PATH 新增 `D:\Program Files\Git\bin` 与 `D:\Program Files\Git\usr\bin`，**且排在 `C:\Windows\System32` 之前**
3. 验证：`which bash` 指向 Git 的 bash.exe

> 注意：不要把这个顺序反过来（System32 在前），否则 `bash` 会回到 WSL、Unix 工具全失效。

### 2. 编码规范（写入 AGENTS.md）

读文件三原则：

```
- 所有可能有中文输出/报错的命令，走 Git Bash 或 PowerShell 通道，不在 cmd 裸跑。
- PowerShell 读文件必须显式指定编码：Get-Content '文件' -Encoding UTF8
- 读文本文件优先用 Git Bash（永远 UTF-8，最稳）：bash -c "cat 'C:/路径/文件'"
```

### 3. 引号策略

```
- 优先两层引号：外层双引号 + 内层单引号。
  bash -c "grep -E '(TODO|FIXME)' /d/project"
- 单引号内需要嵌套单引号：用 '\'' 拼接。
- 真三层嵌套（ssh/docker exec + 复杂正则）→ 用临时脚本文件，不用引号硬拼。
```

## 验证

```bash
which bash                     # 指向 Git Bash
echo "中文测试"                 # 无乱码
bash -c "printf '%s\n' a 'b(c)' | grep '(c)'"   # 引号正常
```
