# 基于年月和人员ID查询已存档社保台账结果

> 接口编号：#76  
> 端点：`POST /openapi/thirdparty/api/insurance/v1/staffMonthlyLedger/list`  
> 鉴权：Bearer access_token

---

## 描述

按年月+员工 ID 列表查询已存档的社保台账结果。**福利管理的完整月度查询接口**——返回所有社保/公积金/其他福利的明细数据。

---

## 请求参数

| 参数 | 位置 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| year | body | Integer | **是** | 福利台账年份 |
| month | body | Integer | **是** | 福利台账月份 |
| staffIdList | body | List\<String\> | **是** | 员工 ID 列表 |

---

## 请求示例

```json
{
  "year": "2024",
  "month": "3",
  "staffIdList": ["bc8b04e4-539d-4cd5-b539-52265ce68bc8"]
}
```

---

## 响应参数

| 参数 | 类型 | 说明 |
|------|------|------|
| code | int | 0=成功 |
| data | Object | 社保台账详情 |

### data 结构

| 字段 | 类型 | 说明 |
|------|------|------|
| year | Integer | 年份 |
| month | Integer | 月份 |
| headerMap | Map\<String, String\> | **字段定义**（code→name 映射，40+ 字段） |
| dataList | Array | **员工台账数据列表** |
| ↳ staffId | String | 员工 ID |
| ↳ dataMap | Map\<String, String\> | 员工台账值（key=headerMap 中的 code，value=实际值） |

### headerMap 常见字段（部分）

| code | name |
|------|------|
| `SI_BENEFIT_NAME` | 社保方案名称 |
| `HF_BENEFIT_NAME` | 公积金方案名称 |
| `SI_PB` | 社保申报基数 |
| `HF_PB` | 公积金申报基数 |
| `SI_TYPE_0001_P` / `_C` | 养老个人 / 单位 |
| `SI_TYPE_0002_P` / `_C` | 医疗个人 / 单位 |
| `SI_TYPE_0003_C` | 工伤单位 |
| `SI_TYPE_0004_P` / `_C` | 大病医疗个人 / 单位 |
| `SI_TYPE_0005_P` / `_C` | 失业个人 / 单位 |
| `SI_TYPE_0006_PB` | 生育申报基数 |
| `HF_TYPE_0001_P` / `_C` | 住房公积金个人 / 单位 |
| `TOTAL` / `TOTAL_P` / `TOTAL_C` | 合计 / 个人合计 / 单位合计 |
| `SI_TOTAL` / `SI_TOTAL_P` / `SI_TOTAL_C` | 社保合计 / 个人 / 单位 |
| `HF_TOTAL` / `HF_TOTAL_P` / `HF_TOTAL_C` | 公积金合计 / 个人 / 单位 |
| `OTHER_TYPE_0001_PB` | 企业年金申报基数 |

---

## 响应示例

```json
{
  "data": {
    "year": 2024,
    "month": 3,
    "headerMap": {
      "SI_BENEFIT_NAME": "社保方案名称",
      "SI_PB": "社保申报基数",
      "SI_TYPE_0001_P": "养老个人",
      "SI_TYPE_0001_C": "养老单位",
      "SI_TYPE_0002_P": "医疗个人",
      "SI_TYPE_0002_C": "医疗单位",
      "SI_TYPE_0003_C": "工伤单位",
      "SI_TYPE_0004_P": "大病医疗个人",
      "SI_TYPE_0004_C": "大病医疗单位",
      "SI_TYPE_0005_P": "失业个人",
      "SI_TYPE_0005_C": "失业单位",
      "HF_BENEFIT_NAME": "公积金方案名称",
      "HF_PB": "公积金申报基数",
      "HF_TYPE_0001_P": "住房公积金个人",
      "HF_TYPE_0001_C": "住房公积金单位",
      "TOTAL": "合计",
      "TOTAL_P": "个人合计",
      "TOTAL_C": "单位合计",
      "SI_TOTAL": "社保合计",
      "HF_TOTAL": "公积金合计"
    },
    "dataList": [
      {
        "staffId": "bc8b04e4-...",
        "dataMap": {
          "SI_PB": "50000.0",
          "SI_TYPE_0001_P": "5000.00",
          "SI_TYPE_0001_C": "5000.00",
          "SI_TYPE_0002_P": "2311.80",
          "SI_TYPE_0002_C": "2311.80",
          "HF_PB": "50000.0",
          "HF_TYPE_0001_P": "300.00",
          "HF_TYPE_0001_C": "2100.00",
          "TOTAL": "19800.76",
          "TOTAL_P": "7661.04",
          "TOTAL_C": "12139.72"
        }
      }
    ]
  },
  "code": 0,
  "message": "SUCCESS",
  "errorResult": false
}
```

---

## 测试结果

**2026-06-17**：端点可达，无 token 返回 401（符合预期）。

---

## 福利管理调用链路

```
#72 社保方案集合 → 查可用方案
#73/#74/#75 基数字段 → 查基数配置
                        ↓
#76 社保台账 → year + month + staffIdList
                ↓
          headerMap(字段定义) + dataList[](员工实际值)
```
