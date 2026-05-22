# Codex Config

> Codex 双轨制配置仓库：OAuth 账号登录 + API Key 代理登录。支持一键恢复。

## 双轨制登录

Codex 支持两种登录方式，可同时配置、互补使用：

| 方式 | 用途 | 配置文件 |
|------|------|----------|
| **OAuth 登录** | Codex 客户端身份认证（ChatGPT 账号） | `~/.codex/auth.json` |
| **API Key 代理** | 实际模型调用（通过本地 CLIProxyAPI） | `~/.codex/config.toml` + `~/.local/cli-proxy-api/config.yaml` |

### 轨一：OAuth 登录（ChatGPT 账号）

通过 ChatGPT 账号 OAuth 登录，获取 `id_token` / `access_token` / `refresh_token`，存储在 `auth.json`。

- 支持 Google / Apple / Microsoft 等第三方 OAuth
- Token 自动刷新
- 多账号池（43+ 账号）通过 CPAMC 管理系统自动切换

### 轨二：API Key 代理登录（CLIProxyAPI）

通过本地 CLIProxyAPI 代理服务器（`127.0.0.1:8317`）转发请求到上游 AI 服务商。

- 兼容 OpenAI Responses API 格式
- 支持多 provider round-robin 路由
- 当前上游：Longcat（`LongCat-2.0-Preview`）、商汤（`deepseek-v4-flash`）

## 快速恢复

### 方式一：curl 一键恢复

```bash
# 恢复 Codex 主配置
curl -fsSL https://raw.githubusercontent.com/skingstyle/codex-config/main/config/codex-config.toml \
  -o ~/.codex/config.toml

# 恢复 CLIProxyAPI 后端配置
curl -fsSL https://raw.githubusercontent.com/skingstyle/codex-config/main/config/cli-proxy-api.yaml \
  -o ~/.local/cli-proxy-api/config.yaml
```

### 方式二：git clone 后手动复制

```bash
git clone https://github.com/skingstyle/codex-config.git
cd codex-config

cp config/codex-config.toml ~/.codex/config.toml
cp config/cli-proxy-api.yaml ~/.local/cli-proxy-api/config.yaml
cp config/hooks.json ~/.codex/hooks.json
```

## 验证

```bash
codex exec --skip-git-repo-check "reply with: ok"
```

看到 `ok` 即成功。报 `404` 说明本地代理服务未启动。

## 目录结构

```
.
├── README.md
├── ABOUT.md
└── config/
    ├── codex-config.toml          # Codex 主配置模板（脱敏）
    ├── cli-proxy-api.yaml         # CLIProxyAPI 后端配置模板（脱敏）
    ├── hooks.json                 # Hooks 配置模板
    ├── auth.json.example          # OAuth auth.json 结构示例
    └── codex-cpa.toml             # 精简版 CPA 配置（兼容旧版）
```

## 配置说明

| 文件 | 路径 | 说明 |
|------|------|------|
| `codex-config.toml` | `~/.codex/config.toml` | 主配置：模型、沙箱、桌面、插件、MCP、环境变量 |
| `cli-proxy-api.yaml` | `~/.local/cli-proxy-api/config.yaml` | 代理后端：providers、API keys、路由策略 |
| `hooks.json` | `~/.codex/hooks.json` | 工具钩子：RTK 代理、OpenWolf 钩子 |
| `auth.json` | `~/.codex/auth.json` | OAuth token（不入库，见 auth.json.example） |

## License

MIT
