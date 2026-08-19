# 根据流程id修改审批流程状态

**接口编号**：#107  
**接口路径**：`POST /openapi/thirdparty/api/workflow/update/process/status`  
**请求方式**：POST  
**鉴权方式**：Bearer Token  
**所属模块**：审批相关  
**创建日期**：2023-09-20  
**创建人**：Beck Wang  

---

## 接口描述

根据流程 id 修改审批流程状态。**仅支持开启了三方审批推送的审批流程**。

---

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 备注 |
|------|------|------|------|------|
| processInfoVo | body | Object | 是 | 请求参数对象 |
| processId | processInfoVo | String | 是 | 流程 id |
| processStatus | processInfoVo | String | 是 | 流程状态：`PASS`（已通过）/ `DENIED`（已驳回） |
| endTime | processInfoVo | Long | 否 | 流程结束时间（毫秒时间戳），不填默认当前时间 |

---

## 响应参数

| 参数 | 类型 | 父节点 | 备注 |
|------|------|--------|------|
| code | String | Response Body | 返回码 |
| message | String | Response Body | 错误信息 |
| url | String | Response Body | - |
| data | String | Response Body | null |
| showType | Long | Response Body | - |
| errorResult | Boolean | Response Body | 是否正确返回 |

---

## 响应示例

```json
{
  "code": 0,
  "message": null,
  "url": null,
  "data": null,
  "showType": 0,
  "errorResult": false
}
```

---

## 注意事项

1. **仅支持开启三方审批推送的审批流程**——未开启的流程调用将返回错误
2. `processStatus` 只支持 `PASS`（已通过）和 `DENIED`（已驳回），不支持作废/撤销等状态
3. `endTime` 为毫秒时间戳，不填则取服务器当前时间
4. 请求体为 `{processId, processStatus, endTime}` 扁平对象，非层层嵌套
