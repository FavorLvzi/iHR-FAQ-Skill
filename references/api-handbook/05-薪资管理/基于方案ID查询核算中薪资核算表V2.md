# 基于方案ID查询核算中薪资核算表V2

> 接口编号：#57  
> 端点：`POST /openapi/thirdparty/api/payroll/v2/salaryTask/querySalary`  
> 鉴权：Bearer access_token  
> 版本：v2

---

## 描述

按薪资方案 ID + 年份/月份，查询核算中的薪资表数据（V2 接口）。**读取接口**，支持按员工和字段筛选。返回每条记录的 `summaryData` Map，其中 code 对应的字段含义需从 #56 的 `salaryFieldList` 获取。

---

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| salaryPlanId | body | Long | **是** | 薪资方案 ID |
| year | body | int | **是** | 薪资所属年份 |
| month | body | int | **是** | 薪资所属月份 |
| page | body | int | **是** | 分页页码（0=第一页） |
| pageSize | body | int | 否 | 分页大小（默认 500） |
| staffIdList | body | List\<String\> | 否 | 员工 ID 列表（不传=查全部） |
| fieldCodeList | body | List\<String\> | 否 | 查询字段 code 列表（不传=查所有字段） |

---

## 请求示例

```json
{
  "salaryPlanId": 1234567,
  "year": 2022,
  "month": 7,
  "page": 0,
  "pageSize": 100,
  "staffIdList": ["11111", "22222"],
  "fieldCodeList": ["code1", "code2"]
}
```

---

## 响应参数

| 参数 | 类型 | 说明 |
|------|------|------|
| code | int | 0=成功 |
| message | String | 状态消息 |
| data | Array | 核算中薪资数据 |

### data[]. 结构

| 字段 | 类型 | 说明 |
|------|------|------|
| staffId | String | 员工 ID |
| staffName | String | 员工姓名 |
| mobileNo | String | 手机号 |
| departmentId | Long | 部门 ID |
| departmentName | String | 部门名称 |
| summaryData | Map\<String, String\> | **薪资字段值 Map**，key=字段 code，value=字段值 |

---

## 响应示例

```json
{
  "code": 0,
  "message": "SUCCESS",
  "data": [
    {
      "staffId": "1212255555454",
      "staffName": "张三",
      "mobileNo": "12345678901",
      "departmentName": "测试部门",
      "departmentId": 121,
      "summaryData": {
        "code1": "1111"
      }
    }
  ],
  "errorResult": false
}
```

---

## 测试结果

**2026-06-17**：端点可达，无 token 返回 401（符合预期）。

---

## 调用链路

```
#55 GET 方案列表 → 拿 planId
#56 POST 方案详情 → 拿 salaryFieldList[]（code→name 映射）
                      ↓
#57 POST 查核算表 → salaryPlanId + year + month
                      ↓
                summaryData → 用 #56 的 code-name 映射解析各字段值
```

---

## 注意事项

1. `summaryData` 中的 key（code）对应 #56 中 `salaryFieldList[].code`
2. 不传 `fieldCodeList` 则返回所有字段，数据量可能很大
3. 不传 `staffIdList` 则返回方案下所有员工
4. `page` 从 0 开始（非 1）
