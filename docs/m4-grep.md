# M4 搜索工具链（rg + grep 排除模式）

> 目标：搜索快、不误伤产物目录、不乱码。核心是"rg 优先 + grep 必须排除"。

## 痛点

- `grep` 不带排除，扫一遍 `node_modules` / `target` / `dist` 能卡 3 分钟，还刷屏。
- Windows 下原生 grep 经常遇到编码问题，中文搜索乱码。
- 搜到的是构建产物 / 第三方依赖的命中，全是噪音。

## 解法一：安装 ripgrep（rg）

rg 多线程、自动跳过 `.gitignore` 匹配的目录（node_modules/dist/target 等），速度快一个数量级。

### 安装

```bash
# macOS
brew install ripgrep

# Windows
winget install BurntSushi.ripgrep.MSVC
# 或
scoop install ripgrep

# Linux
apt install ripgrep     # Debian/Ubuntu
yum install ripgrep     # CentOS/RHEL
```

验证：

```bash
which rg
rg --version
```

## 解法二：grep 排除模式（grep 兜底时必须带）

不是所有环境都有 rg。**只要用 grep，就必须显式排除产物目录**：

```bash
grep -r "keyword" --exclude-dir=node_modules --exclude-dir=target --exclude-dir=dist --exclude-dir=.git
```

写入 AGENTS.md 的规范：

```
## 代码搜索规范（强制）
- grep 一律用 rg（自动跳过 node_modules/dist/target/.git，多线程）。
- 必须用 grep 时，必须加：
  --exclude-dir=node_modules --exclude-dir=target --exclude-dir=dist --exclude-dir=.git
```

## 进阶：Windows 环境变量设置

Windows 上想让 Unix 工具（grep/awk/sed/tail 等）直接在 PowerShell/cmd 里用：

1. 系统 PATH 新增 `D:\Program Files\Git\usr\bin`
2. **排在 `C:\Windows\System32` 之前**（否则 `sort`/`find` 等会被 Windows 版抢走，行为不一致）
3. 验证：`which grep` 指向 Git 的 usr/bin

> ⚠️ 这个顺序很关键：System32 在前会让 `find`、`sort` 等命令变成 Windows 原生版，与 Linux/mac 行为不一致，脚本容易踩坑。

## 验证

```bash
rg "TODO" src/            # 快，且自动跳过产物目录
grep -r "TODO" src/ --exclude-dir=node_modules --exclude-dir=target --exclude-dir=dist --exclude-dir=.git  # 兜底写法
```
