# 通过流程ID、审批节点、审批人、审批状态执行审批任务

**接口编号**：#109  
**接口路径**：`POST /openapi/thirdparty/api/workflow/v1/process/complete/task`  
**请求方式**：POST  
**鉴权方式**：Bearer Token  
**所属模块**：审批相关  
**创建日期**：2024-12-26  
**创建人**：Haoqi Wang  

---

## 接口描述

通过流程 ID、审批节点、审批人、审批状态执行审批任务（模拟审批操作）。

---

## 前置：获取审批节点 ID（nodeId）

- 联系 i人事客服支持获取
- 或从 **发起和审批单据后同步当前流程审批人** 接口的请求 body 中 `extraInfo` 下的 `nodeApproverData`（`Map<String,Object>`，key 为审批节点）获取

---

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 备注 |
|------|------|------|------|------|
| companyId | QueryParam | String | 是 | 公司 id |
| processInfoVo | Body | Object | 是 | 请求参数对象 |
| processId | processInfoVo | String | 是 | 流程 ID |
| nodeId | processInfoVo | String | 是 | 审批节点 |
| staffId | processInfoVo | String | 是 | 审批人 ID（i人事花名册） |
| approveResultType | processInfoVo | String | 是 | 审批状态 |
| approveMsg | processInfoVo | String | 是 | 审批意见 |

---

## approveResultType 枚举

| 值 | 含义 | 备注 |
|----|------|------|
| APPROVE | 同意 | |
| REJECT | 驳回 | |
| BACK | 退回 | 暂只能退回发起人 |

---

## 请求示例

**URL**：
```
POST .../workflow/v1/process/complete/task?companyId=123456
```

**Body**：
```json
{
  "processId": "fdfdf23232323",
  "nodeId": "a_t",
  "staffId": "121212aead",
  "approveResultType": "APPROVER",
  "approveMsg": "同意"
}
```

---

## 响应参数

| 参数 | 类型 | 父节点 | 备注 |
|------|------|--------|------|
| code | String | Response Body | 返回码 |
| message | String | Response Body | 错误信息 |
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

1. `nodeId` 需要提前获取（客服或同步接口），不能随意填写
2. `staffId` 必须是 i人事花名册中的真实员工 ID
3. `BACK`（退回）暂时只支持退回到发起人
4. 需配合 `companyId` 作为 QueryParam
