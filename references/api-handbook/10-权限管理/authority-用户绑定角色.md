# authority 权限域 · 用户绑定角色 + 角色功能授权（用例1 关键）

> **2026-08-18 用户抓包新增**——**"给用户添加角色"的写操作**（bind_role）+ **"给角色分配功能权限"**（authorizationForRole）+ 绑定时的角色候选列表（whileBind）。

## ① 用户绑定角色（bind_role，POST）★用例1 核心

`POST https://v5.ihr360.com/gateway/authority/api/auth/user/bind_role?r_id=&u_id=`

（body 含 webUserId + roleIds，待补 Payload 精确字段）

**响应**：`{code: 0, data: true}`（绑定成功）

## ② 绑定时的角色候选列表（role/list/whileBind/v2，POST）

`POST https://v5.ihr360.com/gateway/authority/api/auth/role/list/whileBind/v2?webUserId={userId}&newAddRoleName=null&r_id=&u_id=`

| 参数 | 说明 |
|------|------|
| `webUserId` | 目标用户 ID（绑给谁） |
| `newAddRoleName` | 绑定时可顺带新建角色（传角色名） |

**响应**（data[] 候选角色）：含系统内置（如 `RECRUIT_ADMIN_ROLE` 招聘超级管理员）+ 自定义；关键字段 `selectedAndNotCancelable`（是否已选且不可取消）。

## ③ 角色功能授权（authorizationForRole，POST）

`POST https://v5.ihr360.com/gateway/authority/api/auth/role_fun/authorizationForRole?r_id=&u_id=`

（body 含 roleId + functionIds 功能权限点，待补 Payload）

**响应**：`{code: 0, data: true}`

## ④ 当前用户权限（老接口 .do）

`GET https://v5.ihr360.com/gateway/web/user/getUserAuthority.do?r_id=&u_id=`（.do 后缀老接口，备用）

## 用户授权完整链路（2026-08-18 已闭环 6/7 步）

```
① existByMobile(手机号)                → 校验
② simpleUser/update(用户信息)          → 创建用户，返回 userId
③ simpleUser/subHr/list               → 管理员列表确认
④ role/list/whileBind/v2(webUserId)    → 绑定时的角色候选列表
⑤ user/bind_role(webUserId+roleIds)    → ★用户绑定角色（data:true）
⑥ role_fun/authorizationForRole        → ★角色功能授权（data:true）
⑦ 数据范围 dataAuths 赋值               → ⏳ 待抓（listByIds 显示 dataAuths 结构 {authorized,hsaAll,it,admin}）
```
