# authority 权限域 · 角色管理（role 系列）

> **2026-08-18 用户抓包新增**——角色 CRUD 相关接口（**用例2「表格建角色」核心**）。域：`/gateway/authority/api/auth/role/...` + `/gateway/authority/aggregate/auth/role/...`，鉴权 = Cookie + XSRF。

## ① 角色名校验（isNameExist，POST）

`POST /gateway/authority/api/auth/role/isNameExist?r_id=&u_id=`（body 含 roleName）

> 预检：角色名是否已存在（同 existByMobile 模式，创建前校验）。

## ② 创建/更新角色（updateRole，POST）★用例2核心

`POST /gateway/authority/api/auth/role/updateRole?r_id=&u_id=`

**响应**（返回创建的角色对象）：

| 字段 | 说明 |
|------|------|
| `id` | 角色 ID（UUID） |
| `roleName` | 角色名（如"ai测试"） |
| `roleCode` | 角色编码（自动生成 UUID） |
| `roleType` | `ROLE_TYPE_USER_DEFINE`（用户自定义）/ `ROLE_TYPE_SYSTEM_ROLE`（系统内置） |
| `innerRole` | 是否内置角色（false=自定义可编辑） |
| `companyId` / `creatorUserId` | 所属公司 / 创建人 |

## ③ 角色列表分页（role/page，POST）

`POST /gateway/authority/aggregate/auth/role/page?page=1&rows=10&r_id=&u_id=`

**响应**（分页 content[]）：

| 字段 | 说明 |
|------|------|
| `id` / `roleName` / `roleCode` | 角色标识 |
| `roleType` | 角色类型（SYSTEM_ROLE 内置 / USER_DEFINE 自定义） |
| `innerRole` / `innerRoleAllowViewFunctionAssign` | 内置角色 / 是否允许查看功能分配 |
| `dataAuthorized` | 是否已分配数据权限 |
| `description` | 角色说明（如"编制管理员，包含编制管理模块的所有数据权限和功能权限"） |

> 实测：共 25 个角色（3 页），含系统内置（如 roleCode=`HC_DEPARTMENT_ADMIN_ROLE` 编制管理员）与用户自定义（`ROLE_TYPE_USER_DEFINE`，如"ai测试"）。

## 用例2（表格建角色）链路

```
① isNameExist(roleName)     → 校验不重名
② updateRole(role信息)      → 创建角色，返回角色 id
③ role/page                → 角色列表确认
④ 后续：给用户绑定该角色 + 数据范围（见 simpleUser/listByIds 的 roleList/dataAuths 字段）
```
