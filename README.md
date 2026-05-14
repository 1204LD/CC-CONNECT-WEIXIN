# Claude Code 接入个人微信配置指南

借助 [cc-connect](https://github.com/chenhg5/cc-connect) 开源项目，将本地 Claude Code 桥接到个人微信，实现手机端远程操控 AI 编程助手。

## 效果

手机微信发送消息 → 本地 Claude Code 处理 → 结果实时返回微信。

## 适用平台

- 个人微信（ilink 协议）
- 无需公网 IP，无需服务器
- Windows 10/11（macOS / Linux 同理）

## 快速开始

详见 [CLAUDE-CODE接入微信配置手册-通用版.md](./CLAUDE-CODE接入微信配置手册-通用版.md)

核心步骤：

1. 从 [GitHub Releases](https://github.com/chenhg5/cc-connect/releases) 下载 Beta 版 `.exe`
2. 创建 `config.toml`
3. 扫码绑定微信
4. 启动服务，从手机微信发送首条消息

## 前置条件

- Node.js 18+
- Claude Code（已配置 `ANTHROPIC_API_KEY`）
- 不需要 Docker

## 本仓库内容

| 文件 | 说明 |
|------|------|
| `CLAUDE-CODE接入微信配置手册-通用版.md` | 完整配置手册，含踩坑记录 |

## 相关链接

- [cc-connect 开源项目](https://github.com/chenhg5/cc-connect)
- [cc-connect 安装指南](https://github.com/chenhg5/cc-connect/blob/main/INSTALL.md)
- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
