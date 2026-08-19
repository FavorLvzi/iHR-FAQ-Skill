# 获取第三方选项数据 GET

| 属性 | 值 |
|------|-----|
| **编号** | #103 |
| **接口类型** | **OA 回调接口**（非 OpenAPI，URL 由第三方提供） |
| **请求方式** | GET |
| **认证方式** | 无（由第三方自行鉴权） |
| **创建人** | Haoqi Wang |
| **创建时间** | 2021-02-24 |

## 功能描述

i人事审批流程中，单选/多选组件可配置"同步第三方选项数据"。i人事 会 **GET 请求**第三方提供的接口地址获取可选下拉项。

⚠️ **这不是 OpenAPI**, 而是 i人事 → 第三方的**回调**。URL 在流程设置的组件配置中填写。

## 请求

请求参数由第三方定义，配置于接口 URL 中（如 `?deptId=xxx&...`）。

## 响应（由第三方返回，必须遵守此格式）

| 参数 | 类型 | 父节点 | 备注 |
|------|------|--------|------|
| code | Integer | Response Body | 返回码：0=OK, -1=INFO, -2=ERROR |
| message | String | Response Body | 错误消息 |
| errorResult | Boolean | Response Body | 是否错误 |
| data | List | Response Body | 选项列表 |
| dataId | String | data | 选项 ID |
| dataName | String | data | 选项名称 |

## 响应示例

```json
{
  "code": 0,
  "message": null,
  "errorResult": false,
  "data": [
    { "dataId": "option1", "dataName": "选项1" },
    { "dataId": "option2", "dataName": "选项2" },
    { "dataId": "option3", "dataName": "选项3" }
  ]
}
```

## 关键约束

- 返回格式**必须严格遵守**文档，格式错误将导致无法正确获取数据
- `code` 为 Integer（非 String），与 OpenAPI 不同
- 此接口无 i人事标准鉴权，第三方需自行处理
