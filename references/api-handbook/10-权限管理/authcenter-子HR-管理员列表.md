# authcenter · 子HR-管理员列表（simpleUser/subHr/list）

> **2026-08-18 用户抓包新增**——**管理员（子HR）列表查询**，授权链路第 3 步。返回当前公司全部子HR/管理员（分页）。

## 端点

`POST https://v5.ihr360.com/gateway/authcenter/delegate/simpleUser/subHr/list`

| 参数 | 位置 | 类型 | 说明 |
|------|------|------|------|
| `r_id` | query | String | 随机 id |
| `u_id` | query | Long | 操作者 uid |
| （body） | — | JSON | 分页参数（page/size 等，待补 Payload） |

## 响应（分页结构）

```json
{
  "code": 0,
  "data": {
    "totalPages": 1,
    "totalElements": 18,
    "content": [ { "id": "xxx", "staffId": "yyy", "staffName": "...", "userName": "..." }, ... ]
  }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `content[].id` | String | 用户 ID（userId） |
| `content[].staffId` | String | 关联员工 ID（**可空**——简单用户可不关联员工） |
| `content[].staffName` / `userName` | String | 姓名 |

## 要点

- **subHr = 子HR = 管理员**：本接口即"管理员列表"查询
- `staffId` 可空：简单用户（只登录不发工资条等）与正式员工（staffId 有值）并存
- 与 `simpleUser/update` 交叉验证：新建用户后立即出现在列表中
- 列表含**无员工关联的用户**（如测试账号、纯管理账号）——这类用户走 `user/me` 的 `authorities` 判断权限
