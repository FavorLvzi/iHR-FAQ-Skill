# manage 域 · 公司已开通应用列表（companyapplication/openedlist）

> **2026-08-18 用户抓包新增**——公司已开通的应用（产品模块）列表。与权限的关系：**每个应用含 `associatedFunctionIds`（关联的功能权限点 ID）**——功能权限的顶层划分。域：`/gateway/manage/aggregate/...`。

## 端点

`GET https://v5.ihr360.com/gateway/manage/aggregate/companyapplication/openedlist?r_id=&u_id=`

## 响应（data[] 应用列表）

| 字段 | 说明 |
|------|------|
| `id` | 应用 ID（如 7=行政） |
| `code` | 应用编码（如 `administrative` / `attendance` / `approval`） |
| `name` | 应用名（行政/考勤/审批…） |
| `description` | 应用描述 |
| **`associatedFunctionIds[]`** | **关联的功能权限点 ID**（与菜单树 `authorityId` 对应，如行政=`1180000`） |
| `enabled` / `opened` | 启用 / 已开通 |
| `provider` / `type` / `typeName` | 提供方 / 类型（IRENSHI 自有应用等） |
| `applyType` | 申请类型（如 ADMINISTRATIVE） |
| `iconName` / `orderNum` | 图标 / 排序 |

## 主要应用（code 对照，2026-08-18 实测）

```
stafforganization 组织人事 / insurancepayroll 薪资福利 / attendance 考勤
kpi 绩效 / smart-kpi 智慧绩效 / approval 审批 / administrative 行政
talent 人才管理 / training 培训 / soke i培训 / ivva_recruit i招聘
report/report_v2 智数 / smartapp 智搭云 / xinshuitong 薪税通
irenshiapp i人事APP / openapi OpenApi / recruit_v2 招聘
digitalcontractqiyusuo 契约锁 / groupsign 团队打卡 / linescheduling 划线排班
```

## 与授权的关系

```
应用（本接口）→ 功能权限点（associatedFunctionIds ↔ 菜单树 authorityId）
→ 角色分配功能权限（role updateRole 时勾选功能）
→ 用户绑定角色（simpleUser/update 带 roleIds）
```

> 给角色/用户分配"功能权限"时，勾选项即来自应用的功能权限点（如行政=1180000）。数据范围（dataAuths）独立于功能权限，另见 simpleUser/listByIds 的 dataAuths 字段。
