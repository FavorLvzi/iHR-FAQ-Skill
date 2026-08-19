# authcenter · 简单用户-创建/更新（simpleUser/update）

> **2026-08-18 用户抓包新增**——授权链路第 2 步：`existByMobile` 返回 false 后，**创建简单用户**；返回新建用户 ID。与 existByMobile 配合完成"新建用户"。

## 端点

`POST https://v5.ihr360.com/gateway/authcenter/delegate/simpleUser/update`

| 参数 | 位置 | 类型 | 说明 |
|------|------|------|------|
| `r_id` | query | String | 随机 id |
| `u_id` | query | Long | 操作者 uid |
| （body） | — | JSON | 用户信息（手机号/用户名等，字段待补全 Payload） |

## 响应

```json
{ "businessErrorCode": null, "code": 0, "data": "b92c085e2af045abaf11a1d7669e1b1b", "errorResult": false }
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data` | String | **新建/更新的用户 ID（userId）** |

## 授权链路（已确认前 3 步）

```
① existByMobile(webUserMobile=手机号)     → data:false（未注册）
② simpleUser/update(用户信息)             → data: 新建 userId  ✅（本接口）
③ simpleUser/subHr/list()                → 管理员列表（含刚创建的 userId，交叉验证）
④ 添加角色 / 数据范围                      → 待继续抓包
```

> **交叉验证**：subHr/list 返回的第 2 条 id = update 返回的 userId（b92c085e...）→ 确认 update 创建的用户已进入子HR（管理员）列表。
