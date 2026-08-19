# manage 域 · 合同公司列表（corporation/getAllCorporationList）

> **2026-08-18 用户抓包新增**——**合同公司（合同主体）完整列表**。与数据授权的 `contractCorpIds` 对应（值=本接口 `id` UUID）。比 `component/.../Corporation/getALL`（仅 id+companyName）**多含 `departmentIds` 关联**——可做"公司→部门"映射。

## 端点

`POST https://v5.ihr360.com/web/corporation/getAllCorporationList`

## 响应（data[] 合同公司）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String | **合同公司 ID（UUID）——数据授权 contractCorpIds 的值** |
| `companyName` | String | 公司名 |
| `companyId` | String | 所属公司（主公司 ID） |
| **`departmentIds[]`** | List | **关联的部门 ID 列表**（如 Albuquerquee=[98,104,101]；主公司=[10,43,4,28,31,22,25,19,7,13,16]）——合同公司与部门的关系 |
| `staffIds[]` | List | 关联员工 |
| `status` | String | 认证状态：`NOT_VERIFIED`（未认证）/ `VERIFY_FAILURE`（认证失败）等 |
| `isApplyToAll` | Boolean | 是否适用全部 |
| `organizationChartId` / `parent` | String | 组织架构图 / 父级 |
| `taxNo` / `taxArea` / `phone` / `bankAccount` / `address` | String | 税号/税区/电话/银行账号/地址 |
| `userId` / `createdDate` / `isDeleted` | — | 创建人 / 创建时间 / 删除标记 |

## 与数据授权的关系

```
contractCorpIds（数据授权字段）= 本接口 id（UUID）
departmentIds（本接口）= 该公司覆盖的部门 → "按公司授权"可推导覆盖哪些部门
```

> 两个合同公司接口对比：
> - `web/corporation/getAllCorporationList`：全字段（含 departmentIds 关联）——**合同公司管理/展示**
> - `component/api/organization/v1/Corporation/getALL`：仅 id+companyName——数据授权选择器轻量版
