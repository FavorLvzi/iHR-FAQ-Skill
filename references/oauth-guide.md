# iHR360 OpenAPI 授权指南（OAuth client_credentials）

> 供 AI 在使用本 skill 时**临时**调用 iHR360 OpenAPI 获取数据。完整流程已写入 SKILL.md「首次使用：API 授权」；本文为详细规格与代码示例。

---

## 安全提醒（向客户说明）

- **AppID / AppSecret 是客户公司的敏感凭证**，只应出现在客户自己的环境与授权流程中。
- 本 skill 不内置任何人的凭证；**每次在新环境首次使用时，要求客户提供自己的 AppID 和 AppSecret**。
- 凭证不写入任何客户报告 / HTML / 回答；不跨环境共享凭证文件。

---

## 获取 Access Token

**请求**：

```
POST https://openapi.ihr360.com/openapi/oauth/token
Authorization: Basic base64(AppID:AppSecret)
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&scope=client
```

**Python 示例**：

```python
import base64, json, urllib.request
from urllib.parse import urlencode

APP_ID = "客户提供的 AppID"
APP_SECRET = "客户提供的 AppSecret"

auth_str = base64.b64encode(f"{APP_ID}:{APP_SECRET}".encode()).decode()
body = urlencode({"grant_type": "client_credentials", "scope": "client"}).encode()
req = urllib.request.Request(
    "https://openapi.ihr360.com/openapi/oauth/token",
    data=body,
    headers={
        "Authorization": f"Basic {auth_str}",
        "Content-Type": "application/x-www-form-urlencoded",
    },
    method="POST",
)
with urllib.request.urlopen(req, timeout=15) as resp:
    data = json.loads(resp.read().decode())
    token = data.get("access_token") or data.get("data", {}).get("access_token")
```

**响应**：

```json
{
  "code": 0,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 7200
  }
}
```

---

## 携带 Token 调 API

```
GET https://openapi.ihr360.com/openapi/thirdparty/api/staff/v1/staffs/ids?staffStatus=IN_SERVICE&pageNo=1
Authorization: Bearer {access_token}
Content-Type: application/json; charset=UTF-8
```

---

## 注意事项

| 事项 | 说明 |
|------|------|
| 有效期 | 2 小时，超时后重新获取 |
| 限速 | 10 req/s，批处理时串行 + 重试 |
| 凭证来源 | 客户首次使用时提供，写入工作区 `.workbuddy/openapi-credentials.txt` |
| 凭证文件格式 | 第一行 `APP_ID=xxx`，第二行 `APP_SECRET=xxx` |
| 获取凭证 | 客户在 iHR360 开放平台（open.ihr360.com）→ 应用管理中创建/查看 |
| 安全 | Token 与 AppSecret 仅用于调接口，不写入任何交付物 |

---

## ★ API 实操陷阱速查（2026-08-10 宿主 AI 实测沉淀）

OAuth 授权流程本身准确（实测 62 名在职员工 ID 正常返回），但调用 OpenAPI 接口时会遇到 4 个常见"陷阱"（**不是 skill 的问题，而是 API 文档与实际行为的细微差异**），详见 `references/api-traps.md`：

1. **域混淆**：Base URL 必须是 `openapi.ihr360.com`（非 `api.ihr360.com`）；二方 Gateway 接口用 `v5.ihr360.com`
2. **#106 companyId 无权访问**：应用未授权该公司时报"无权访问的公司id"——检查应用管理中"可访问公司"配置
3. **#106 statuses 数组只能单值**：传 `["PASS", "DENIED"]` 触发 JSON 解析错误，必须 `statuses: ["PASS"]` 单元素
4. **路径版本混淆**：员工详情 V1 `/staff/v1/staffs/{id}/detail`，银行卡 V2 `/staff/v2/staffBank/list`——**同模块不同子模块可能混合 v1/v2**，调前查 `api-handbook/index.md`

---

## 与登录态 / 二方接口的关系

- OpenAPI（Bearer）与 Gateway（Cookie+XSRF）**路径不互通**：`/openapi/...` 必须用 Bearer，`/web/gateway/...` 用宿主登录态，混用 401。
- 二方接口（#116/#117 等）在有宿主登录态时作为补充（额度流水/锁定账本），不是主路径。
- **先授权走 OpenAPI（主路径）→ 二方接口补充 → 降级回答**，顺序不可颠倒。
