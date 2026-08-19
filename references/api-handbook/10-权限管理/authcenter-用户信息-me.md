# authcenter 权限中心 · 当前用户信息（user/me）

> **2026-08-18 用户抓包新增**——`/gateway/authcenter/` 是权限中心域（用户授权相关接口所在），本接口为**当前登录用户完整信息**（含角色权限）。鉴权 = Cookie + XSRF（前端二方接口），宿主环境可尝试 `IHR360_API_TOKEN` Bearer。

## 端点

`GET https://v5.ihr360.com/gateway/authcenter/delegate/user/me`

| 参数 | 位置 | 类型 | 说明 |
|------|------|------|------|
| `r_id` | query | String | 随机 id（防缓存） |
| `u_id` | query | Long | 用户 uid |

## 响应关键字段（脱敏示例）

| 字段 | 类型 | 说明 |
|------|------|------|
| `accountId` / `accountIdStr` | Long/String | 账号 ID |
| `accountName` | String | 账号名（手机号） |
| `mobileNo` | String | 手机号 |
| `username` / `staffName` | String | 用户名 / 员工姓名 |
| `userId` | String | 用户 UUID |
| `staffId` | String | 员工 UUID |
| `email` | String | 邮箱 |
| `locked` / `activated` | Boolean | 锁定 / 激活状态 |
| **`authorities[]`** | List | **角色权限**：`ROLE_HR_ADMIN`（HR管理员）/ `ROLE_USER` / `ROLE_STAFF` / `ROLE_HR` |
| `currentCompany` | Object | 当前公司（companyId/companyName/hrContactName/uid/staffId/authorities/status/versionCode/companyAccountType 等） |
| `boundCompanyList[]` | List | **绑定的公司列表**（多公司场景：每个公司独立 companyId/uid/staffId/authorities） |
| `hr` / `bound` / `admin` | Boolean | HR / 已绑定 / 管理员 标志 |
| `irenshiAdmin` / `subHr` | Boolean | 总公司管理员 / 子HR 标志 |
| `staff` | Boolean | 员工标志 |
| `uid` | Long | 用户 uid（同 u_id 参数） |
| `attributes` | Object | accountName/username 等冗余 |

## 使用场景

- **查当前用户权限**：authorities（角色）+ admin/irenshiAdmin/subHr（权限标志）→ 判断提问人是否有 HR 管理权限（配合「视角识别」）
- **多公司切换**：boundCompanyList 每个公司独立 authorities → 当前提问人在哪个公司是什么角色
- **authcenter 域探索起点**：用户授权（添加管理员/角色/数据范围）的写操作接口大概率同域（`/gateway/authcenter/...`），待继续抓包
