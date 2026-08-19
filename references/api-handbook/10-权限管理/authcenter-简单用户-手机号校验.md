# authcenter · 简单用户-手机号存在校验（simpleUser/existByMobile）

> **2026-08-18 用户抓包新增**——授权链路第 1 步：添加用户到管理员前，**校验手机号是否已存在**。返回 `data: false` = 手机号未注册（可新建用户）；`true` = 已存在（走已有用户绑定）。

## 端点

`POST https://v5.ihr360.com/gateway/authcenter/delegate/simpleUser/existByMobile`

| 参数 | 位置 | 类型 | 说明 |
|------|------|------|------|
| `webUserMobile` | query | String | 手机号（web 用户） |
| `r_id` | query | String | 随机 id（防缓存） |
| `u_id` | query | Long | 操作者 uid |

## 响应

```json
{ "code": 0, "message": null, "data": false, "showType": 0, "errorResult": false }
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `data` | Boolean | `false`=手机号未注册（可新建） / `true`=已存在（绑定已有用户） |

## 使用场景（用户授权链路）

```
① existByMobile(webUserMobile=手机号)  → false：新建简单用户；true：绑定已有用户
② 后续：添加管理员 / 添加角色 / 添加数据范围（待继续抓包，同 authcenter 域）
```

> 前置：authority 域 `getUserMenuTree`（定位授权入口）+ authcenter `user/me`（当前用户权限）。授权页面路由 `account-v2/user-auth?menu=140&subMenu=14002&thirdMenu=1400200`。
