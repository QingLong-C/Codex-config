# About Codex Config

## 项目背景

Codex 是 OpenAI 推出的 AI 编程桌面客户端。默认使用 OpenAI 官方 API，但通过配置可以接入任意兼容 OpenAI API 格式的自定义后端。

本仓库记录了完整的 **双轨制配置方案**：既保留 ChatGPT OAuth 登录的便利性，又通过本地代理层灵活接入多个上游 AI 服务商。

## 架构概览

```
┌─────────────────────────────────────────────────────────┐
│                    Codex 桌面客户端                       │
│                                                         │
│  ┌─────────────┐    ┌──────────────────────────────┐   │
│  │  auth.json   │    │        config.toml            │   │
│  │  (OAuth)     │    │  model_provider = "CPA"       │   │
│  │  ChatGPT账号 │    │  base_url = 127.0.0.1:8317    │   │
│  └──────┬──────┘    └──────────┬───────────────────┘   │
│         │                      │                        │
│         │    ┌─────────────────┘                        │
│         │    │                                           │
│         ▼    ▼                                           │
│  ┌──────────────────────────────────────────────────┐   │
│  │              CLIProxyAPI 代理层                    │   │
│  │              127.0.0.1:8317                       │   │
│  │                                                    │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  │   │
│  │  │  Longcat   │  │   商汤     │  │  (更多...) │  │   │
│  │  │  LongCat   │  │  deepseek  │  │            │  │   │
│  │  │  2.0-Prev  │  │  -v4-flash │  │            │  │   │
│  │  └────────────┘  └────────────┘  └────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                         │
│  路由策略: round-robin                                   │
│  OAuth 账号池: 43+ Codex 账号 (CPAMC 管理)              │
└─────────────────────────────────────────────────────────┘
```

## 两种登录方式详解

### OAuth 登录（ChatGPT 账号）

Codex 客户端通过标准 OAuth 2.0 流程登录 ChatGPT 账号：

1. 用户在 Codex 桌面端选择登录方式（Google / Apple / Microsoft）
2. 完成 OAuth 跳转后，Codex 获取 JWT token 三元组
3. Token 存储在 `~/.codex/auth.json`
4. CLIProxyAPI 的 `auth-dir` 目录下可存放多个 OAuth 账号文件
5. CPAMC 管理系统负责：额度监控 → 自动切换 → 401 重登

**auth.json 结构：**
```json
{
  "auth_mode": "chatgpt",
  "tokens": {
    "id_token": "eyJ...",
    "access_token": "eyJ...",
    "refresh_token": "rt.xxx",
    "account_id": "uuid"
  },
  "last_refresh": "ISO-8601-timestamp"
}
```

### API Key 代理登录（CLIProxyAPI）

CLIProxyAPI 是一个 Go 编写的代理服务器，将多种 AI 服务商的 API 统一包装为 OpenAI 兼容格式：

- **上游无关性**：支持 OpenAI、Longcat、商汤、Gemini、Claude 等
- **多账号轮换**：支持 round-robin / 优先级路由
- **OAuth 集成**：可以为每个上游配置独立的 OAuth 账号
- **管理面板**：内置 Web UI (`/management`) 查看额度、切换账号

**当前上游配置：**

| Provider | Base URL | 模型 |
|----------|----------|------|
| Longcat | `https://api.longcat.chat/openai` | `LongCat-2.0-Preview` |
| 商汤 | `https://token.sensenova.cn/v1` | `deepseek-v4-flash`, `sensenova-6.7-flash-lite` |

## config.toml 关键配置说明

### 模型与提供商

```toml
model = "LongCat-2.0-Preview"    # 使用的模型
model_provider = "CPA"           # 对应 [model_providers.CPA]

[model_providers.CPA]
base_url = "http://127.0.0.1:8317/v1"
experimental_bearer_token = "sk-xxx"  # 代理层 API key
requires_openai_auth = true
```

### 安全策略

```toml
approval_policy = "never"           # 不询问权限
sandbox_mode = "danger-full-access" # 完全文件系统访问
```

### 环境变量注入

```toml
[shell_environment_policy.set]
OPENAI_API_KEY = "sk-xxx"
LONGCAT_API_KEY = "ak-xxx"
SILICONFLOW_API_KEY = "sk-xxx"
DISABLE_AUTOUPDATER = "1"          # 禁用自动更新
```

### 插件系统

Codex 通过插件扩展能力：

| 插件 | 用途 |
|------|------|
| `browser@openai-bundled` | 浏览器自动化 |
| `chrome@openai-bundled` | Chrome 扩展集成 |
| `computer-use@openai-bundled` | 计算机控制 |
| `documents@openai-primary-runtime` | 文档处理 |
| `github@openai-curated` | GitHub 集成 |
| `spreadsheets@openai-primary-runtime` | 电子表格 |
| `presentations@openai-primary-runtime` | 演示文稿 |

### MCP 服务器

```toml
[mcp_servers.node_repl]
command = "/Applications/Codex.app/Contents/Resources/node_repl"
```

### Hooks 系统

通过 `hooks.json` 在工具调用前后注入自定义逻辑：

| 触发点 | 钩子 | 作用 |
|--------|------|------|
| PreToolUse (Bash) | `rtk hook` | 命令走 RTK 代理省 token |
| PreToolUse (Read) | `pre-read.js` | 预读钩子 |
| PreToolUse (Write) | `pre-write.js` | 预写钩子 |
| SessionStart | 3 个钩子 | 会话初始化（脚本 + JS） |
| UserPromptSubmit | `wolf-context-hint.sh` | 上下文提示 |
| Stop | 2 个钩子 | 会话清理 |

## CLIProxyAPI 后端配置

`config.yaml` 核心结构：

```yaml
host: "127.0.0.1"
port: 8317
auth-dir: "~/.cli-proxy-api"     # OAuth 账号文件目录
routing:
  strategy: "round-robin"        # 路由策略
api-keys:
  - sk-xxx                       # 代理层 API key

openai-compatibility:
  - name: ProviderName
    priority: 1
    base-url: https://...
    api-key-entries:
      - api-key: xxx
    models:
      - name: model-name
```

## CPAMC 多账号管理

CPAMC（Codex Pool Account Management & Control）是一套运行在 macOS 菜单栏的账号管理系统：

- **Swift 编写的菜单栏应用**：实时监控所有账号额度
- **自动切换**：当前账号额度耗尽时自动切换到下一个
- **401 重登**：检测到未自动重登时自动重新登录
- **线程恢复**：切换账号后自动恢复之前的对话线程
- **GitHub 同步**：配置自动备份到 GitHub

## 如何贡献

1. Fork 本仓库
2. 修改配置文件模板（注意脱敏）
3. 提交 PR

## 相关资源

- [CLIProxyAPI 上游项目](https://github.com/router-for-me/CLIProxyAPI) — 34k+ stars
- [OpenWolf](https://github.com/QingLong-C/openwolf) — AI agent 上下文管理
