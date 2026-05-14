# Claude Code 接入个人微信 配置手册

> cc-connect v1.3.3-beta.2 | 2026-05-14 实测通过 | 适用于 Windows 10/11

---

## 项目简介

借助 [cc-connect](https://github.com/chenhg5/cc-connect) 开源项目，将本地 Claude Code 桥接到个人微信，实现手机端远程操控 AI 编程助手。

**核心原理**：手机微信发消息 → 腾讯 ilink 服务器中转 → 本地 cc-connect 接收 → 调用 Claude Code → 结果原路返回微信。

---

## 前提条件

- Windows 10/11
- Node.js 18+（已安装）
- Claude Code（已安装并配置好 `ANTHROPIC_API_KEY`）
- 不需要 Docker

---

## 安装步骤

### 1. 下载 cc-connect

浏览器打开 https://github.com/chenhg5/cc-connect/releases

下载最新 **Beta 版**（必须 Beta，稳定版无个人微信支持）的 `cc-connect-*-windows-amd64.exe`，重命名为 `cc-connect.exe`，放到某个目录（如 `D:\tools\`），将该目录加入系统 PATH。

```powershell
# 以 D:\tools 为例，替换为你的实际路径
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";D:\tools", "User")
```

重新打开终端，验证：
```powershell
cc-connect --version
```

### 2. 创建配置文件

创建 `config.toml`（路径可自选，以下以 `D:\cc-connect-config\` 为例）：

```toml
language = "zh"

[log]
level = "info"

[[projects]]
name = "wechat-ai"

  [projects.agent]
  type = "claudecode"

  [projects.agent.options]
  work_dir = "D:/your-project-path"      # 替换为你的项目绝对路径
  mode = "default"

  [[projects.platforms]]
  type = "weixin"

  [projects.platforms.options]
  token = "SCAN_NEEDED"
```

> `token` 必须填一个非空占位值，否则启动报错 `token is required`。扫码后会自动替换为真实 token。

### 3. 扫码绑定（关键步骤）

扫码命令**必须使用默认配置路径**（`%USERPROFILE%\.cc-connect\config.toml`），指定 `-config` 参数会导致二维码不弹出。

```powershell
# 创建默认目录
mkdir %USERPROFILE%\.cc-connect

# 将你的配置文件复制到默认路径
copy <你的配置路径>\config.toml %USERPROFILE%\.cc-connect\config.toml

# 扫码（二维码显示在终端）
cc-connect weixin setup --project wechat-ai
```

用手机微信扫描终端显示的二维码，确认授权。

### 4. 同步配置

扫码成功后 token 写入默认路径，同步回你的自定义路径：

```powershell
copy %USERPROFILE%\.cc-connect\config.toml <你的配置路径>\config.toml
```

此时 `config.toml` 中会新增 `base_url` 和 `account_id` 字段，token 也已替换为真实值。

### 5. 启动服务

```powershell
cc-connect -config <你的配置路径>\config.toml
```

正常日志：
```
level=INFO msg="platform ready" project=wechat-ai platform=weixin
level=INFO msg="cc-connect is running" projects=1
```

### 6. 首次发消息

手机微信给绑定的微信号发一条消息（如"你好"），完成 `context_token` 缓存。之后即可正常对话。

---

## 最终配置示例

扫码完成后，`config.toml` 大致如下：

```toml
language = "zh"

[log]
level = "info"

[[projects]]
name = "wechat-ai"

  [projects.agent]
  type = "claudecode"

  [projects.agent.options]
  work_dir = "D:/your-project-path"
  mode = "default"

  [[projects.platforms]]
  type = "weixin"

  [projects.platforms.options]
  token = "xxx@im.bot:xxxx"              # 扫码自动填入
  base_url = "https://ilinkai.weixin.qq.com"
  account_id = "xxx@im.bot"              # 扫码自动填入
```

---

## 开机自启（可选）

### 启动脚本

创建 `.bat` 脚本（路径自定）：

```bat
@echo off
chcp 65001 >nul
title cc-connect
start "cc-connect" /MIN cc-connect -config <你的配置路径>\config.toml
timeout /t 3 >nul
exit
```

### 注册到 Windows 启动项

```powershell
$WshShell = New-Object -ComObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\cc-connect.lnk")
$Shortcut.TargetPath = "<你的脚本路径>\cc-connect-start.bat"
$Shortcut.WorkingDirectory = "<脚本所在目录>"
$Shortcut.WindowStyle = 7
$Shortcut.Save()
```

取消自启：删除 `%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\cc-connect.lnk`。

---

## 日常使用

**手动启动**：
```powershell
cc-connect -config <配置路径>\config.toml
```

**停止**：在 cc-connect 终端窗口按 `Ctrl+C`。

**聊天命令**：

| 命令 | 功能 |
|------|------|
| `/new` | 新会话 |
| `/list` | 查看会话列表 |
| `/switch <ID>` | 切换会话 |
| `/stop` | 停止当前执行 |
| `/mode yolo` | 自动批准所有操作 |
| `allow` / `允许` | 批准当前权限请求 |

**权限模式**：

| 模式 | 行为 |
|------|------|
| `default` | 每次工具调用需手动批准 |
| `acceptEdits` | 文件编辑自动批准 |
| `plan` | 只做规划不执行 |
| `yolo` | 所有操作自动批准 |

---

## 踩坑记录

1. **必须 Beta 版**：稳定版没有 weixin 平台，运行时报 `unknown platform "weixin"`。
2. **推荐二进制安装**：直接从 GitHub Releases 下载 `.exe`，避免 `npm install -g` 影响已有的 Claude Code 安装。
3. **文件名要重命名**：下载的 `.exe` 含版本号（如 `cc-connect-v1.3.3-beta.2-windows-amd64.exe`），需改为 `cc-connect.exe`。
4. **首次 token 不能为空**：必须填占位值（如 `SCAN_NEEDED`），否则启动报 `weixin: token is required`。
5. **扫码必须用默认路径**：`weixin setup` 带 `-config` 参数时不弹二维码。需先将配置文件复制到 `%USERPROFILE%\.cc-connect\config.toml` 再执行扫码。
6. **扫码完要重启**：token 写入配置文件后需重新启动 cc-connect 才能生效。
7. **首条消息初始化**：扫码启动后必须从微信发送一条消息，才能完成 `context_token` 缓存绑定。

---

## 注意事项

- **电脑需保持开机**：cc-connect 是本地运行的桥接服务，电脑关机或休眠后微信端将无法回复。
- **API Key 沿用已有配置**：cc-connect 不管理 API Key，直接读取当前系统的 `ANTHROPIC_API_KEY` 环境变量。
- **微信消息长度限制**：过长回复可能被截断，可在指令中加"请精简回答"。
- **个人微信功能处于 Beta 阶段**：偶尔可能出现稳定性问题，建议关注 [cc-connect GitHub Issues](https://github.com/chenhg5/cc-connect/issues) 获取最新进展。

---

> 2026-05-14 实测通过 | 基于 cc-connect v1.3.3-beta.2 + Claude Code 2.1.132
