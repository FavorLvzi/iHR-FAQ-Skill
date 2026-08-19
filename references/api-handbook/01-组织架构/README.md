# 组织架构模块

> 所属：iHR360 OpenAPI 参考手册 — 01-组织架构  
> 接口范围：#9 ~ #25  
> 来源：OpenAPI 官方文档 + Gateway 实际验证  
> 更新日期：2026-06-17

---

## 模块说明

组织架构模块涵盖以下核心业务实体，覆盖从底层基础数据到上层组织分类的完整链路：

```
职务分类 (#24)               部门 (#9, #13, #14)       法律实体 (#22)
    ↓                            ↓                       ↓
职务 (#19)                   工作地点 (#23)
    ↓
职位 (#15, #16, #17)
    ↓
职级 (#20)
    ↓
体系表 (#21): 序列 → 子序列 → 职层 → 职级
    ↓
多维组织 (#25): 维度 → 组织部门
```

**模块特点**：
- **部门**：支持分页搜索（#9）、批量查询（#13）、字段元数据查询（#14）
- **职位**：三个查询入口（#15/#16/#17），各接口 positionState 格式不一致需注意
- **职级体系**：职务→职位→职级三层结构 + 体系表（序列→子序列→职层→职级）
- **基础数据**：法律实体、工作地点、职务分类为组织架构的支撑字典
- **多维组织**：独立路径，按维度 ID 查询

---

## 接口索引

| # | 接口名称 | 路径 | 方式 | 鉴权 | 用途 |
|---|---------|------|------|------|------|
| 9 | 获取部门清单v3 | `/openapi/thirdparty/api/org/v1/organizations/search` | POST | Bearer token | 分页查部门，支持按名称/编码/状态筛选 |
| 10 | 批量创建部门 | `/openapi/thirdparty/api/org/v1/organizations/create` | POST | Bearer token | 批量创建部门（写入接口） |
| 11 | 更新部门 | `/openapi/thirdparty/api/org/v1/organizations/{departmentId}/update` | POST | Bearer token | 更新部门名称/编码/地点/编制/负责人等 |
| 12 | 设置部门负责人 | `/openapi/thirdparty/api/org/v1/organizations/principal/set` | POST + Query | Bearer token | 设置/更改部门负责人 |
| 13 | 部门详情（批量） | `/openapi/thirdparty/api/org/v1/organizations/list` | POST | Bearer token | 按部门 ID 数组批量获取部门详情 |
| 14 | 获取部门字段设置 | `/openapi/thirdparty/api/org/v1/metadata` | GET | Bearer token | 查询部门字段定义/类型/必填状态等元数据 |
| 15 | 获取公司职位清单V2 | `/openapi/thirdparty/api/org/v1/organizations/positions/search` | POST | Bearer token | 分页查职位，支持按编码/名称筛选 |
| 16 | 获取部门下的职位清单 | `/openapi/thirdparty/api/org/v1/organizations/positions` | GET + Query | Bearer token | 按部门 ID 获取该部门所有职位（无分页） |
| 17 | 根据职位id获取职位清单 | `/openapi/thirdparty/api/org/v1/organizations/positionIds` | POST | Bearer token | 按职位 ID 数组批量获取职位详情 |
| 18 | 根据职位id获取适用职级 | `/openapi/thirdparty/api/org/v1/organizations/positions/getPositionGrade` | POST | Bearer token | 按职位 ID 查适用职级（返回 Map 格式） |
| 19 | 获取职务清单 | `/openapi/thirdparty/api/org/v1/organizations/jobTitles` | GET | Bearer token | 获取全公司所有职务清单 |
| 20 | 获取职级清单 | `/openapi/thirdparty/api/org/v1/organizations/positiongrades` | GET | Bearer token | 获取全公司所有职级清单 |
| 21 | 获取体系表清单 | `/openapi/thirdparty/api/org/v1/gradesystem/all` | GET | Bearer token | 获取全公司职级体系表 |
| 22 | 获取法律实体清单v3 | `/openapi/thirdparty/api/org/v1/corporations/search` | POST | Bearer token | 分页搜索法律实体 |
| 23 | 获取工作地点清单 | `/openapi/thirdparty/api/org/v1/companysites` | GET | Bearer token | 获取全公司工作地点清单 |
| 24 | 获取职务分类清单 | `/openapi/thirdparty/api/org/v1/jobCategory/get` | GET | Bearer token | 获取全公司职务分类清单 |
| 25 | 查询多维组织 | `/openapi/thirdparty/api/dimensionOrganization/v1/dimension/organizations` | GET + Query | Bearer token | 按维度 ID 查询多维组织部门清单 |

---

## 鉴权说明

所有组织架构模块接口均使用 **Bearer token** 鉴权（OpenAPI 标准）。  
需先通过 AppID+AppSecret 获取 access_token，然后在请求头中携带 `Authorization: Bearer {token}`。

---

## 文件清单

| 文件 | 包含接口 | 说明 |
|------|---------|------|
| `README.md` | 全部索引 | 模块说明 + 接口索引表 |
| `部门.md` | #9, #13, #14 | 部门查询/批量详情/字段元数据 |
| `职位.md` | #15, #16, #17, #18 | 职位搜索/部门下职位/按ID查/适用职级 |
| `职务-职级-体系表.md` | #19, #20, #21 | 职务/职级/体系表 |
| `法律实体-工作地点-职务分类.md` | #22, #23, #24 | 法律实体/工作地点/职务分类 |
| `多维组织.md` | #25 | 多维组织查询 |
