# M5 按需代理（pi-p 包装命令）

> 目标：访问 GitHub / 国外资源需要代理，但**不想全局代理**、**不想所有 Pi 进程都走代理**——按需开启。

## 痛点

- 国内访问 GitHub / 搜索国外资源频繁失败，但开全局代理又拖慢国内站点、影响公司内网。
- Pi 的模型请求、`fetch`、`curl`、`npm` 子进程都可能要访问国外，需要代理。
- 但**不是每次都要代理**：大部分时候访问国内/内网，代理反而碍事。

## 解法：pi-p 包装命令（按需启动）

建一个 `pi-p` 命令：**写入代理环境变量 → 传参启动 Pi**。想走代理时用 `pi-p`，不想时用 `pi`，互不影响。

```
普通模式    pi            → 不走代理（国内/内网正常访问）
代理模式    pi-p          → 走代理（访问 GitHub / 国外搜索/下载）
```

### Windows（`pi-p.cmd`，放到 PATH 目录）

> 文件名即命令名：存成 `pi-p.cmd` 才能用 `pi-p` 调用；存成别的名字（如 `piproxy.cmd`）就得用那个名字调用。

```bat
@echo off
setlocal
set "HTTP_PROXY=http://127.0.0.1:7890"
set "HTTPS_PROXY=http://127.0.0.1:7890"
set "NO_PROXY=localhost,127.0.0.1,::1"
pi.cmd %*
exit /b %ERRORLEVEL%
```

使用：

```powershell
pi-p
pi-p --continue   # 带参数启动
```

`setlocal` 保证代理环境变量**只在 Pi 进程及其子进程生效**，退出即失效，不改系统、不改当前终端。

### macOS / Linux（`pi-p` shell 函数，写入 `~/.zshrc` 或 `~/.bashrc`）

```bash
pi-p() {
  HTTP_PROXY=http://127.0.0.1:7890 \
  HTTPS_PROXY=http://127.0.0.1:7890 \
  NO_PROXY=localhost,127.0.0.1,::1 \
  pi "$@"
}
```

使用：

```bash
pi-p
pi-p --continue
```

> 端口 `7890` 是示例，换成你代理软件的 HTTP / Mixed 端口（Clash 默认 7890、V2Ray 10809 等）。

## 进阶：全局配置 httpProxy（可选，与 pi-p 二选一）

如果希望**所有 Pi 项目/会话都走代理**，可以直接配置全局。编辑 `~/.pi/agent/settings.json`：

```json
{
  "httpProxy": "http://127.0.0.1:7890"
}
```

它会把 `HTTP_PROXY` / `HTTPS_PROXY` 应用到 Pi 进程及其子进程，不开系统代理。

> ⚠️ **两个方案二选一，不要同时配**：`httpProxy` 是"全都要"，`pi-p` 是"按需要"。
> 一旦配了 `httpProxy`，普通 `pi` 也会走代理，`pi-p` 就失去了存在意义。按需场景请只保留 `pi-p`。

## 相关细节（来自实战讨论）

- **NO_PROXY 很重要**：本地模型、数据库、本地 HTTP 服务别走代理，保留 `localhost,127.0.0.1,::1`。
- **子进程也会继承**：Pi 启动的 `bash`/`curl`/`npm`/部分 `git` 请求会继承代理环境变量——这仍然是进程级，不是系统全局。
- **外部 MCP / 独立服务不一定读到**：如果 `search`/`fetch` 是独立 Node/Python 服务，可能需要在该服务单独配代理或调用时传 `proxy` 参数。
- **不要把代理写进项目 AGENTS.md**：那是模型指令文件，不是网络配置文件。

## 验证

```bash
# 代理模式下
pi-p 启动后，让 Pi 执行：
curl -I https://github.com    # 应返回 200

# 普通模式不受影响
pi 启动后：
curl -I https://github.com    # 走默认网络
```

## 引导 skill 的落地方式

检测到用户未配置按需代理时：
1. 询问代理地址与端口（如 `127.0.0.1:7890`），一次问清
2. Windows：生成 `pi-p.cmd` 到 PATH 目录（如 Git 的 usr/bin 或用户目录），文件名必须与要调用的命令名一致
3. macOS/Linux：生成 `pi-p` 函数追加到 `~/.zshrc` / `~/.bashrc`
4. 验证：`pi-p` 后 `curl -I https://github.com` 返回 200
