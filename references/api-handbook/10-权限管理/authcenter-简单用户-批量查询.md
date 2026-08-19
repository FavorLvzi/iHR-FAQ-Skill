# authcenter · 简单用户-按ID批量查询（simpleUser/listByIds）

> **2026-08-18 用户抓包新增**——按用户 ID 批量查用户详情。**关键：响应含 `roleList`（角色）+ `dataAuths`（数据权限）字段**——判断用户"已有哪些角色/数据范围"的查询接口。

## 端点

`POST https://v5.ihr360.com/gateway/authcenter/delegate/simpleUser/listByIds?r_id=&u_id=`

（body 含 userId 列表）

## 响应（data[] 数组）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String | 用户 ID |
| `staffId` / `staffName` | String | 关联员工（可空） |
| `userName` | String | 用户名 |
| `mobileNo` | String | 手机号 |
| `isAdmin` | Boolean | 是否管理员 |
| `isLock` | Boolean | 是否锁定 |
| **`roleList` / `roleListStr`** | List/String | **已分配的角色**（★角色权限） |
| **`dataAuths` / `dataAuthStr`** | List/String | **已分配的数据权限**（★数据范围） |
| `platformRole` | String | 平台角色 |
| `editable` / `staticUser` | Boolean | 可编辑 / 静态用户 |
| `lastLoginTime` / `loginTimes` | 时间/Int | 最后登录 / 登录次数 |
| `email` / `remark` | String | 邮箱 / 备注 |

## 使用场景

- **查用户已有哪些角色/数据权限**：roleList + dataAuths（授权前先查现状）
- **授权链路现状查询**：确认"添加管理员/角色/数据范围"后的最终状态

## ★ roleList 结构（2026-08-18 实测，bind_role 传参依据）

`roleList` 是**角色绑定详情数组**（非纯 roleIds），每项：

| 字段 | 说明 |
|------|------|
| `roleId` | **★角色 ID（UUID）——bind_role 的 roleIds 参数就是取此项** |
| `roleName` | 角色名（超级管理员/ai测试/淘金Admin…） |
| `roleCode` | 角色编码：**内置角色=语义码**（MASTER/SMART_APP/HC_*/RECRUIT_*）/ **自定义角色=UUID** |
| `companyId` / `userId` | 所属公司 / 用户 |
| `autoBind` | 是否自动绑定 |

**内置角色编码参考（2026-08-18 实测）**：

| roleCode | roleName | 说明 |
|----------|---------|------|
| `MASTER` | 超级管理员 | 全权限，`dataAuths={authorized:true, hsaAll:true, admin:true}` |
| `SMART_APP` | 智搭云管理员 | |
| `HC_DEPARTMENT_ADMIN_ROLE` | 编制管理员 | |
| `RECRUIT_ADMIN_ROLE` | 招聘超级管理员 | |

> `dataAuths` 结构：`{authorized（已授权）, hsaAll（全部数据）, admin（管理员）, it（IT）}`——超级管理员 hsaAll=true。

## 授权链路全景（2026-08-18 已确认）

```
查入口： authority getUserMenuTree（定位授权页面）
当前用户：authcenter user/me（角色/管理员标志）
新建用户：existByMobile（校验）→ simpleUser/update（创建，返回 userId）
管理员列表：simpleUser/subHr/list
用户详情：simpleUser/listByIds（含 roleList + dataAuths）
角色管理：authority isNameExist → updateRole（创建角色）→ role/page（列表）
数据范围：dataAuths 赋值接口（待继续抓包，预期 simpleUser/update 带 dataAuths 或独立 scope 接口）
```
