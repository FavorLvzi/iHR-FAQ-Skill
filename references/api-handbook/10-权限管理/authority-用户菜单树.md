# authority 权限域 · 用户菜单树（webUser/getUserMenuTree）

> **2026-08-18 用户抓包新增**——`/gateway/authority/` 权限域（功能权限）。本接口返回**当前用户可见的完整菜单树**（= 被授权的功能权限），鉴权 = Cookie + XSRF（前端二方）。

## 端点

`GET https://v5.ihr360.com/gateway/authority/aggregate/webUser/getUserMenuTree`

| 参数 | 位置 | 类型 | 说明 |
|------|------|------|------|
| `r_id` | query | String | 随机 id（防缓存） |
| `u_id` | query | Long | 用户 uid |

## 响应结构（data.menu[] 递归树）

| 字段 | 类型 | 说明 |
|------|------|------|
| `key` | Long/String | 菜单/权限点 ID |
| `name` | String | 菜单名（业务名，如"管理员权限"） |
| `url` | String | 菜单路由（如 `account-v2/user-auth?menu=140&subMenu=14002`） |
| `module` | String | 模块标识（pluginMount=插件等） |
| `nav[]` | List | 子菜单（递归） |
| `menuLayer` | Integer | 层级（2=二级） |
| `menuSequence` | String | 排序 |
| **`authorityId`** | String | **权限点 ID**（菜单项对应的功能权限标识） |
| `menuIcon` / `menuOpenType` / `showInSiteMap` | — | 图标/打开方式/站点地图 |

## 权限管理模块在菜单树中的位置（★授权入口）

```
账号(140, module=account)   ← ★一级菜单：账号/权限管理所在
  └── 权限设置 → 权限管理
        ├── 管理员权限    url: account-v2/user-auth?menu=140&subMenu=14002&thirdMenu=1400200
        ├── 员工端权限    （员工 APP 可见功能配置）
        └── 权限分组      （审批流权限分组配置）
```

## 一级菜单映射（2026-08-18 实测，key=菜单ID）

| key | name | module | 说明 |
|-----|------|--------|------|
| 101 | 工作台 | workplatform | |
| 102 | 组织 | company-settings | |
| 103 | 员工 | roster | |
| 108 | 考勤 | attendance-vacation | |
| 105 | 薪资 | cnb-salary | |
| 107 | 社保 | cnb-benefit | |
| 115 | 绩效 | kpi | |
| 176 | 电子签约 | digitalContract | |
| 181 | 智慧绩效 | smart-kpi | |
| 40000 | i招聘 | RECRUIT_IVVA | |
| 111/117/119/133 | 培训系列 | training | |
| 20230531 | 人才管理 | talent | |
| 112 | 行政 | administrative | |
| 213 | 智数 | report | |
| **131** | **审批** | **workflow** | |
| **140** | **账号** | **account** | **★权限管理所在** |
| 930 | 系统 | system | |
| 940/950 | 平台/智能中心 | platform | |

> 每个菜单项含 `authorityId`（权限点 ID，如 1160000/1180000）+ `nameKey`（i18n key）+ `menuSequence`（排序）。

## 使用场景

- **判断用户功能权限**：用户可见的菜单 = 被授权的功能（权限验证）
- **定位授权入口 URL**：权限管理菜单的 url 即"用户授权"页面路由（配合 authcenter 域写操作）
- **菜单树是权限过滤后的结果**：不同角色返回不同菜单（ROLE_HR_ADMIN 全量 vs 员工端精简）

## 权限域全景（2026-08-18 已确认两个域）

| 域 | 接口 | 用途 |
|----|------|------|
| `/gateway/authcenter/` | `delegate/user/me` | 当前用户信息（authorities 角色/admin 标志） |
| `/gateway/authority/` | `aggregate/webUser/getUserMenuTree` | 当前用户功能权限（菜单树） |

> 用户授权**写操作**（添加管理员/添加角色/添加数据范围）待继续抓包（预期仍在 authcenter 或 authority 域）。
