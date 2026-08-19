# 薪资核算根据审批id查询审批文件地址

> 接口编号：#58  
> 端点：`GET /openapi/thirdparty/api/payroll/v1/salaryTask/getFileUrlByProcessId`  
> 鉴权：Bearer access_token

---

## 描述

通过审批 ID 查询薪资核算数据的审批文件地址。**读取接口**，返回文件下载 URL。可用于流程台账导出插件——获取工资表等审批文件的下载链接。

---

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| processId | Query | String | **是** | 审批 ID |

---

## 请求示例

```
GET /payroll/v1/salaryTask/getFileUrlByProcessId?processId=11111
```

---

## 响应参数

| 参数 | 类型 | 说明 |
|------|------|------|
| code | int | 0=成功 |
| message | String | 状态消息 |
| data | String | **审批文件地址 URL**（⚠️ 文档参数表误写为 Boolean，实际为 String） |
| errorResult | Boolean | 错误时=true |

---

## 响应示例

```json
{
  "code": 0,
  "message": "SUCCESS",
  "data": "文件地址url",
  "errorResult": false
}
```

---

## 测试结果

**2026-06-17**：端点可达，无 token 返回 401（符合预期）。

---

## 对流程台账导出插件的价值

此接口可配合 `ihr360-flow-export` 技能使用——在导出审批流程时，如果某个流程是薪资核算审批，可以通过此接口拿到工资表文件的下载地址。
