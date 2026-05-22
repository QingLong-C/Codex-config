# auth.json 结构说明
# 路径: ~/.codex/auth.json
# ⚠️ 此文件包含敏感 token，切勿提交到 git
#
# - id_token: JWT 格式，解码后可获取用户 email/plan_type/user_id
# - access_token: 短期有效（通常 1 小时），用于实际 API 请求
# - refresh_token: 长期有效，access_token 过期后用于获取新的 token
# - account_id: ChatGPT 账号唯一标识
#
# CLIProxyAPI 的 OAuth 账号文件（~/.cli-proxy-api/codex-*.json）
# 结构类似，额外包含 email、expired、disabled、type 等字段

{
  "auth_mode": "chatgpt",
  "OPENAI_API_KEY": null,
  "tokens": {
    "id_token": "YOUR_ID_TOKEN",
    "access_token": "YOUR_ACCESS_TOKEN",
    "refresh_token": "YOUR_REFRESH_TOKEN",
    "account_id": "YOUR_ACCOUNT_ID"
  },
  "last_refresh": "2026-01-01T00:00:00Z"
}
