# 获取第三方选项数据 POST

| 属性 | 值 |
|------|-----|
| **编号** | #104 |
| **接口类型** | **OA 回调接口**（非 OpenAPI，URL 由第三方提供） |
| **请求方式** | POST |
| **Content-Type** | application/json |
| **认证方式** | 无（由第三方自行鉴权） |
| **创建人** | Haoqi Wang |
| **创建时间** | 2021-11-18 |

## 功能描述

与 #103 相同场景，区别在于 **POST 方式**，i人事会将表单中已填写的数据**作为请求体**传给第三方，第三方可根据这些数据**动态过滤**返回的选项。

⚠️ **这不是 OpenAPI**, 而是 i人事 → 第三方的**回调**。

## 请求体

请求体为当前 OA 表单中固定组件的**已填入值**（即组件标记码确认的参数内容）。以加班申请为例：

```json
{
  "total": "2",
  "dateFormat": "yyyy-MM-dd HH:mm",
  "applyReason": "测试加班",
  "overtimeType": "NORMAL",
  "startTime": "2021-02-24 18:00",
  "endTime": "2021-02-24 20:00",
  "overtimeApplyCompensateType": "TRANSFER_TO_REST"
}
```

请求体字段取决于表单设计，**非固定结构**。

## 响应格式

与 #103 完全一致：

```json
{
  "code": 0,
  "message": null,
  "errorResult": false,
  "data": [
    { "dataId": "option1", "dataName": "选项1" },
    { "dataId": "option2", "dataName": "选项2" }
  ]
}
```

| code | 含义 |
|------|------|
| 0 | OK |
| -1 | INFO |
| -2 | ERROR |

## 与 #103 的差异

| 维度 | #103 GET | #104 POST |
|------|----------|-----------|
| 请求方式 | GET | POST |
| 入参 | URL 参数 | JSON Body（表单当前数据） |
| 场景 | 静态选项 | **动态过滤**（根据已填字段返回候选） |
| 响应格式 | 相同 | 相同 |
