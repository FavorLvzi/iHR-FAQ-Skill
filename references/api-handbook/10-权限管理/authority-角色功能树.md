# authority 权限域 · 角色功能树（role_fun/v2/tree，★查角色已分配功能）

> **2026-08-18 用户抓包确认**——**查询角色分配了哪些功能**（与 `authorizationForRole` 写操作配对：读=tree / 写=authorizationForRole）。返回功能树 + 每节点 `check` 标记（是否已分配）。

## 端点

`GET https://v5.ihr360.com/gateway/authority/api/auth/role_fun/v2/tree?roleId={roleId}`

| 参数 | 说明 |
|------|------|
| `roleId` | 角色 ID（UUID，来自 role/page 或 updateRole 返回） |

## 响应结构（data：APP / PC 双端）

```json
{
  "code": 0,
  "data": {
    "APP": { "maxDepth": 5, "tree": [...] },   // 员工端功能树
    "PC":  { "maxDepth": 5, "tree": [...] }    // PC 端功能树
  }
}
```

### tree 节点结构

| 字段 | 说明 |
|------|------|
| `authorityId` | **功能权限点 ID**（如 "200" 应用中心 / "11202013" 入职登记表管理） |
| `authorityName` | 功能名 |
| `parentAuthorityId` | 父级（"0"=根） |
| `authorityCategory` | 类别（如 "B"） |
| `functionCode` | 功能编码（如 `appCenter` / `app.staffManage.entryRegisterManage`） |
| `functionLayer` | 层级（1=一级） |
| **`check`** | **★是否已分配该功能**（true=角色已勾选） |
| `children[]` | 递归子节点 |

## 使用场景（★角色功能权限读写闭环）

```
读：role_fun/v2/tree?roleId=      → 角色已分配哪些功能（check 标记）
写：role_fun/authorizationForRole → 保存角色功能分配（body 含 roleId + functionIds）
```

- **查角色能看哪些模块**：遍历 tree，收集 check=true 的 authorityId → 汇总功能清单
- **修改角色功能**：读现状（tree）→ 改 check → authorizationForRole 保存
- 与 `authcenter` 用户侧关系：用户绑定角色（bind_role）→ 该角色功能（role_fun）→ 用户实际可用功能 = 角色功能 ∩ 用户数据权限（departIds 等）

## 关键认知（功能权限体系）

```
角色功能（role_fun tree）→ 限定"能看哪些模块"（功能列）
用户数据权限（userDataAuth）→ 限定"能看哪些员工的数据"（数据行）
两者独立，用户最终权限 = 角色功能 ∩ 数据权限
```
