# Codex Config

> Codex 双轨制配置仓库：OAuth 账号登录 + API Key 代理登录。

## 双轨制登录

| 方式 | 用途 | 配置文件 |
|------|------|----------|
| **OAuth** | Codex 客户端身份认证（ChatGPT 账号） | `~/.codex/auth.json` |
| **API Key 代理** | 实际模型调用（本地 CLIProxyAPI → 上游服务商） | `~/.codex/config.toml` + `~/.local/cli-proxy-api/config.yaml` |

- **OAuth**：ChatGPT 账号登录，token 自动刷新，多账号池自动切换
- **API Key 代理**：CLIProxyAPI 监听 `127.0.0.1:8317`，兼容 OpenAI Responses API，round-robin 路由多上游

## 一键恢复

### 方式一：Claude Code 一句话（推荐）

直接对 Claude Code 说：

> codex 重装了，帮我恢复自定义 API

Claude Code 会自动从本仓库下载配置、写入 `~/.codex/config.toml`、验证连接。全程 10 秒。

其他触发语：
- "帮我配置 codex 的 CPA 自定义 API"
- "去 GitHub 给 codex 配置自定义 api"

### 方式二：curl 一键恢复

```bash
# 恢复 Codex 主配置
curl -fsSL https://raw.githubusercontent.com/QingLong-C/codex-config/main/config/codex-config.toml \
  -o ~/.codex/config.toml

# 恢复 CLIProxyAPI 后端配置
curl -fsSL https://raw.githubusercontent.com/QingLong-C/codex-config/main/config/cli-proxy-api.yaml \
  -o ~/.local/cli-proxy-api/config.yaml
```

### 方式三：git clone

```bash
git clone https://github.com/QingLong-C/codex-config.git
cd codex-config
cp config/codex-config.toml ~/.codex/config.toml
cp config/cli-proxy-api.yaml ~/.local/cli-proxy-api/config.yaml
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
├── ABOUT.md                  # 架构详解 + config 逐项说明
└── config/
    ├── codex-config.toml      # 主配置模板（脱敏，需替换 PLACEHOLDER）
    ├── cli-proxy-api.yaml     # 代理后端配置模板（脱敏）
    ├── hooks.json             # Hooks 配置模板
    ├── auth.json.example      # OAuth auth.json 结构说明
    └── codex-cpa.toml         # 精简版（兼容旧版）
```

## License

MIT
