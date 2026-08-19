# Gateway 接口

> 接口 #5~#8 + #111~#117 — Gateway 内网接口，二方路径（与 OpenAPI 不同域）。

---

## 鉴权

二方 Gateway 接口在不同环境下鉴权方式不同：

- **前端插件上下文（浏览器）**：Cookie+XSRF（前端插件自动携带）
- **iHR 内置 AI 宿主环境**：读取环境变量 `IHR360_API_TOKEN`，调用时 `Authorization: Bearer {IHR360_API_TOKEN}`，Base URL `https://v5.ihr360.com`，路径前缀 `/web/gateway/...`
- **OpenAPI 的 OAuth Bearer 不互通**：不可用 `openapi.ihr360.com` 的 OAuth token 调本路径（403），也不要把本路径的 token 用于 OpenAPI

> 详细调用方式见 SKILL.md「宿主登录态」小节。

---

## #5 花名册员工搜索（Gateway）

> **路径**：`POST /web/gateway/roster/aggregate/v1/staffs/search`  
> **鉴权**：前端插件 Cookie+XSRF｜宿主 AI 环境：`Authorization: Bearer ${IHR360_API_TOKEN}`（环境变量自动注入，Base URL `v5.ihr360.com`）  
> **状态**：✅ 已验证

### 请求体

```json
{
  "query": {
    "staffStatus": "IN_SERVICE",
    "departmentName": "技术部",
    "staffName": ""
  },
  "page": 1,
  "pageSize": 20
}
```

#### 请求参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `query.staffStatus` | string | 否 | 员工状态筛选：`IN_SERVICE`（在职）、`LEAVE`（离职） |
| `query.departmentName` | string | 否 | 部门名称模糊匹配 |
| `query.staffName` | string | 否 | 员工姓名模糊匹配 |
| `page` | integer | 是 | 页码，从 1 开始 |
| `pageSize` | integer | 是 | 每页条数 |

### 响应结构

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100,
    "page": 1,
    "pageSize": 20,
    "list": [
      {
        "staffName": "张三",
        "mobileNo": "138****1234",
        "staffImageId": "image-uuid",
        "departmentName": "技术部",
        "staffStatus": "IN_SERVICE",
        "enrollInDate": "2023-06-01",
        "lastWorkDay": null
      }
    ]
  }
}
```

### 字段说明

| 字段 | 类型 | 说明 | 验证状态 |
|------|------|------|---------|
| `staffName` | string | 员工姓名 | ✅ 已验证 |
| `mobileNo` | string | 手机号（脱敏） | ✅ 已验证 |
| `staffImageId` | string | 头像图片 ID | ✅ 已验证 |
| `departmentName` | string | 部门名称 | ✅ 已验证 |
| `staffStatus` | string | 员工状态：`IN_SERVICE` / `LEAVE` | ✅ 已验证 |
| `enrollInDate` | string | 入职日期（yyyy-MM-dd） | ✅ 已验证 |
| `lastWorkDay` | string\|null | 最后工作日（离职员工有值） | ✅ 已验证 |
| `staffNo` | string | 工号 | 🔴 未验证 |
| `positionName` | string | 岗位名称 | 🔴 未验证 |
| `staffType` | string | 员工类型 | 🔴 未验证 |
| `probationStatus` | string | 试用期状态 | 🔴 未验证 |
| `probationEndDate` | string | 试用期结束日期 | 🔴 未验证 |

> **🔴 注意**：标有「未验证」的字段可能在请求中声明后导致整个查询被拒绝（返回错误或空结果），使用前需在实际环境中验证。

---

## #6 员工花名册详情批量（Gateway）

> **路径**：`POST /gateway/hr/core/api/employee/detail/list`  
> **鉴权**：前端插件 Cookie+XSRF｜宿主 AI 环境：`Authorization: Bearer ${IHR360_API_TOKEN}`（环境变量自动注入，Base URL `v5.ihr360.com`）  
> **状态**：⚠️ 待确认

### ⚠️ 说明

- **请求参数和返回字段格式尚未确认**，以下为推测结构，使用前需实测验证。
- 推测请求体为员工 ID 列表或 JSON 对象。

#### 推测请求体

```json
{
  "staffIds": ["staff-id-1", "staff-id-2"]
}
```

#### 推测响应

```json
{
  "code": 0,
  "message": "success",
  "data": [
    {
      "staffId": "staff-id-1",
      "staffName": "张三",
      "departmentName": "技术部",
      "positionName": "高级工程师"
    }
  ]
}
```

---

## #7 离职清单（Gateway）

> **路径**：`POST /gateway/hr/core/api/dimission/list`  
> **鉴权**：前端插件 Cookie+XSRF｜宿主 AI 环境：`Authorization: Bearer ${IHR360_API_TOKEN}`（环境变量自动注入，Base URL `v5.ihr360.com`）  
> **状态**：✅ 已验证

### 请求体

```json
{
  "page": 1,
  "pageSize": 20
}
```

### 响应结构

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 50,
    "page": 1,
    "pageSize": 20,
    "list": [
      {
        "staffName": "李四",
        "lastWorkDay": "2026-06-01",
        "confirmQuitDate": "2026-05-15",
        "staffStatus": "LEAVE"
      }
    ]
  }
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `staffName` | string | 员工姓名 |
| `lastWorkDay` | string | 最后工作日（yyyy-MM-dd） |
| `confirmQuitDate` | string | 确认离职日期（yyyy-MM-dd） |
| `staffStatus` | string | 员工状态，离职为 `LEAVE` |

---

## #8 年假报表（Gateway）

> **路径**：`POST /gateway/attendance/leave/api/annual/report/summary`  
> **鉴权**：前端插件 Cookie+XSRF｜宿主 AI 环境：`Authorization: Bearer ${IHR360_API_TOKEN}`（环境变量自动注入，Base URL `v5.ihr360.com`）  
> **状态**：⚠️ 待确认

### ⚠️ 说明

- **请求参数和返回格式尚未确认**，以下为推测结构，使用前需实测验证。
- 推测支持按年份、部门等条件查询年假汇总数据。

#### 推测请求体

```json
{
  "year": 2026,
  "departmentName": "技术部",
  "page": 1,
  "pageSize": 20
}
```

#### 推测响应

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "total": 100,
    "page": 1,
    "pageSize": 20,
    "list": [
      {
        "staffName": "张三",
        "departmentName": "技术部",
        "annualTotalDays": 15,
        "annualUsedDays": 5,
        "annualRemainingDays": 10
      }
    ]
  }
}
```
