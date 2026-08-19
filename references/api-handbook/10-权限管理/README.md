# 10-权限管理（API 手册）

> 2026-08-18 用户抓包沉淀——iHR360 **权限三层模型**：用户 → 人员数据权限 / 用户绑定角色 → 角色功能权限。
> 接口均为二方（Cookie+XSRF，宿主可试 IHR360_API_TOKEN Bearer），无 # 编号（官方未开放 OpenAPI）。

## ★★ 权限三层模型（理解本模块的核心）

```
用户（simpleUser：手机号登录账号，可关联员工 staffId 或纯账号）
│
├── ① 人员数据权限（row_data/userDataAuth）——"能看见哪些员工的数据"
│     字段：departIds（部门）/ contractCorpIds（合同公司）/ siteIds（工作地点）
│           / includeGroups·excludeGroups（员工分组包含·排除）
│
├── ② 绑定角色（user/bind_role）——"用户是哪个角色"
│     字段：roleIds（角色ID列表，来自 roleList[].roleId）
│
└── ③ 角色 → 功能权限（role_fun/authorizationForRole）——"角色能看哪些模块"
      字段：functionIds（功能权限点，来自 authorityTree/菜单 authorityId）
```

**三层职责**：① 限"数据行"（看谁的）· ② 定身份 · ③ 限"功能列"（看哪个模块）——**①和③是独立维度，互不替代**。

## 接口清单（12 文档 / 19 接口，2026-08-18 全部抓包）

| 环节 | 接口 | 文档 |
|------|------|------|
| 当前用户权限 | `delegate/user/me` | authcenter-用户信息-me.md |
| 手机号校验 | `delegate/simpleUser/existByMobile` | authcenter-简单用户-手机号校验.md |
| 创建/更新用户 | `delegate/simpleUser/update` | authcenter-简单用户-更新创建.md |
| 管理员列表 | `delegate/simpleUser/subHr/list` | authcenter-子HR-管理员列表.md |
| 用户详情（角色+数据权限） | `delegate/simpleUser/listByIds` | authcenter-简单用户-批量查询.md |
| 功能菜单树 | `aggregate/webUser/getUserMenuTree` | authority-用户菜单树.md |
| 角色 CRUD | `api/auth/role/isNameExist` + `updateRole` + `aggregate/auth/role/page` | authority-角色管理.md |
| 用户绑定角色 | `api/auth/user/bind_role` + `role/list/whileBind/v2` | authority-用户绑定角色.md |
| 角色功能授权 | `api/auth/role_fun/authorizationForRole` | authority-用户绑定角色.md |
| 数据授权读写 | `api/auth/row_data/userDataAuth`（写）+ `aggregate/webUser/dataAuth`（读） | authority-用户数据授权.md |
| 已开通应用 | `manage/aggregate/companyapplication/openedlist` | manage-公司应用列表.md |
| 合同公司 | `web/corporation/getAllCorporationList`（全）/ `component/.../Corporation/getALL`（简） | manage-合同公司列表.md |
| 部门树 | `component/api/organization/v1/treeSelector` | authority-用户数据授权.md（数据源） |
| 当前用户权限（老） | `web/user/getUserAuthority.do` | authority-用户绑定角色.md（备用） |

## 授权操作链路（用例1：为用户授权）

```
① existByMobile(手机号) → false 未注册
② simpleUser/update(用户信息) → userId
③ subHr/list → 管理员列表确认
④ role/list/whileBind/v2(webUserId) → 候选角色
⑤ user/bind_role(webUserId + roleIds) → 绑定角色
⑥ role_fun/authorizationForRole(roleId + functionIds) → 角色功能权限
⑦ row_data/userDataAuth(webUserId + departIds/contractCorpIds/...) → 人员数据权限
```

> 数据源：部门选项 `component/api/organization/v1/treeSelector`（dwAgentKey）；合同公司 `Corporation/getALL`（UUID）。
> Payload 精确字段待用户抓包补充（bind_role/userDataAuth/update 请求体）。
