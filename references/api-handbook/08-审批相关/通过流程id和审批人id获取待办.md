# 通过流程id和审批人id获取待办

| 属性 | 值 |
|------|-----|
| **编号** | #105 |
| **接口地址** | `POST https://openapi.ihr360.com/openapi/thirdparty/api/workflow/v1/process/task/backLog/{processId}` |
| **请求方式** | POST |
| **认证方式** | Bearer Token |
| **创建人** | Haoqi Wang |
| **创建时间** | 2022-11-10 |

## 功能描述

根据审批流程 ID 和待办人 ID 列表获取**待办链接**（PC端 + H5端），常用于消息推送场景。

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 备注 |
|------|------|------|------|------|
| companyId | Query | String | 是 | 公司ID |
| processId | Path | String | 是 | 审批流程ID |
| staffIds | Body | List\<String\> | 是 | 待办人ID列表 |

**请求体为 JSON 数组**：

```json
["0cbb0a47-c393-4ad2-bba1-19092f1437b8"]
```

## 响应参数

| 参数 | 类型 | 父节点 | 备注 |
|------|------|--------|------|
| code | String | Response Body | 返回码 |
| message | String | Response Body | 错误消息 |
| data | List | Response Body | - |
| errorResult | Boolean | Response Body | - |
| initiator | String | data | 待办发起人ID |
| staffId | String | data | 待办人ID |
| web_url | String | data | PC 待办链接（URL编码） |
| h5_url | String | data | H5 待办链接（URL编码） |
| formBrief | String | data | 摘要信息 |

## 响应示例

```json
{
  "code": 0,
  "message": null,
  "data": [
    {
      "initiator": "c919a869-fb99-4740-9446-42a3ce853e41",
      "staffId": "0cbb0a47-c393-4ad2-bba1-19092f1437b8",
      "web_url": "%2Fproxy%2Fess%2Fmessage-v1%3FsubModule%3Ddetail%26id%3D...",
      "h5_url": "%2Fproxy%2Fh5%2Fmessage-v1%3FsubModule%3Ddetail%26id%3D...",
      "formBrief": "单行文字:111"
    }
  ],
  "errorResult": false
}
```

## 关键说明

| 点 | 说明 |
|------|------|
| URL 编码 | `web_url`/`h5_url` 均为 URL 编码的**相对路径**，使用时需拼接域名 |
| 待办人 | staffIds 列表中的每个用户各返回一条待办记录 |
| 场景 | 典型用于获取审批待办入口链接，配合消息推送跳转 |
