# 字段公式 / 值的获取来源约定（硬规则）

回答"这个字段怎么算的 / 值从哪来"时，**先判断字段类型，再决定去哪取**，绝不能混源、不能瞎编公式。

> **公式长什么样**（词法/语法/70+ 函数签名/字段引用规则 `表名$字段名`/典型场景）：见 `references/formula-language.md`（iHR360 公式语言手册，合并自旧页面生成型 skill）。

## 一、系统内置字段

例：应出勤 / 实际出勤（小时·天数）、迟到、早退、缺勤、缺卡、补卡、加班、各类假期……

| 要取什么 | 去哪取 |
|---------|--------|
| **计算逻辑 / 公式 / 口径** | **只走文档（help）**。API 不返回公式，只能拿到最终计算值。 |
| **实际计算值** | **走 API**（#93 日报 / #94 月报 的标准字段）。 |

> API 手册位置：`api-handbook/07-考勤管理/`

## 二、自定义字段

企业管理员在报表里自建的字段（带公式，引用标准计算项）。

| 要取什么 | 去哪取 |
|---------|--------|
| **公式 (formula)** | **走 API**：#96 获取日报自定义字段、#102 获取月报自定义字段（类 Excel 公式，明文可读） |
| **实际值** | **走 API**：#93/#94 主数据里的 `dailyCustomField` / `periodCustomField` / `periodCustomFieldInfo`，或 #100 获取日报自定义字段值（独立平铺接口） |

## 三、关键认知："标准字段被改写"是什么

- API 返回的 `actualAttendanceHours` 等**永远是系统算好的标准值**。
- 所谓"标准字段被改写"，是指企业在报表里**另建了一个自定义字段**去引用 / 重算它（如用同名 `actualAttendanceHours` 覆盖系统字段）。
- 因此：想知"标准实际出勤小时数怎么来" → 查 help（文档口径）；想知"你们报表里那个实际出勤字段怎么来" → 调 #96/#102 读 `formula`。

## 四、开放给二次加工的系统字段（仅 3 个）

官方明确开放供自定义字段引用的系统字段只有：

- `actualAttendanceHours`（实际出勤小时数）
- `lateMinutes`（迟到分钟数）
- `earlyMinutes`（早退分钟数）

其余系统字段不被开放引用。

## 五、考勤日月报字段 API 链路

| 编号 | 接口 | 路径 | 作用 |
|------|------|------|------|
| #93 | 获取日报 V4 | `POST /openapi/thirdparty/api/tm/v4/daily/reports/search` | 标准字段 + `dailyCustomField`(值) + `signBlockList`(班段签到) |
| #94 | 获取月报 V3 | `POST /openapi/thirdparty/api/tm/v3/period/monthly/reports/search` | 标准字段 + `periodCustomField` + `periodCustomFieldInfo`(值) |
| #96 | 获取日报自定义字段 | `GET /openapi/thirdparty/api/tm/v1/custom/field/list/search` | 返回 `id`/`name`/`formula`/`valueType` 等 |
| #100 | 获取日报自定义字段值 | `POST /openapi/thirdparty/api/tm/v1/daily/custom/search` | 平铺的 `dailyCustomField`，上限 100人×31天 |
| #102 | 获取月报自定义字段 | `GET /openapi/thirdparty/api/tm/v1/period/custom/field/list/search` | 月报维度 `formula`，变量带 `ATT_PERIOD_BASIC$` 前缀 |

```
#96/#102 取 formula(id→name/公式)
        ↓ 解析"实际出勤小时数"等是否被自定义改写
#93/#94 取标准字段值 + 自定义字段值
#100 单独拉日报自定义字段实际值（与 #96 的 id 对应）
```

> 月报没有独立的"字段值"接口——月报自定义字段值嵌在 #94 的 `periodCustomFieldInfo`。
> 复杂场景规则设置（休班次等）是系统配置界面逻辑，API 手册无对应接口，只能靠 #96/#102 的 `formula` 反推或去 help 查。

## 六、薪资字段（★ 与考勤机制根本不同，2026-08-06 实测确认）

**薪资字段没有"系统内置逻辑"**——每个薪资项都是公式定义的（企业配置），这一点与考勤完全相反：

| 维度 | 考勤（#93/#94） | 薪资（#53~#71） |
|------|----------------|----------------|
| 字段机制 | 有系统内置字段（标准值系统算好） | **无内置逻辑，全部公式定义** |
| 字段定义 | #96/#102 返回 `formula`（自定义可读） | 二方 `companySalaryfieldMetas/list` 每项带 `formula` 明文 |
| 系统项目 | 有标准计算项 | 二方列表含 `systemsalaryfieldCode`；OpenAPI #68 只有 code/name/type |
| 值 | #93/#94 直接返回标准值+自定义值 | #67 返回 91 项值（`{'v':'xxx'}`） |
| 公式解读 | 内置走文档；自定义走 API formula | **二方 `companySalaryfieldMetas/list` = 公式来源**（★2026-08-06 抓包确认） |

**实测结论（2026-08-06，员工A 2025-08）**：
- #67 拿到 91 个字段值（`basesalary=20000.00` / `positionsalary=30000.00` / `grossincome=18260.87` / `taxablesalary=11170.25` / `salarytax=335.11`），值格式 `{'v':'xxx'}`
- ★ **二方 `POST /gateway/payroll/api/companySalaryfieldMetas/list` 返回 185 个薪资项目，每项带 `formula` 明文**（如 `=STAFF_ROSTER$positiveDate`、`=CHOOSE(MATCH(...)...)`）——**公式来源打通**
- 公式 token 解析：`GET /gateway/component/manage/api/component/table/manage/enable?code=payroll.table.companysalaryfield`（exprItems）
- OpenAPI 侧（#56/#64/#68/#69）无公式——OpenAPI 只做"值验证"

**薪资公式解读的方法论（问题4 完整闭环）**：
1. **公式内容** → 二方 `companySalaryfieldMetas/list`（每项 `formula` 明文，185 项全量）
2. **公式解析** → 二方 `manage/enable`（exprItems token 流，可选校验）
3. **语法解读** → `references/formula-language.md`（函数/字段引用 `表名$字段`）
4. **值验证** → OpenAPI #67（按员工+方案+年月，91 项值）——formula 中的变量 → #67 值 → 代入验证
5. **防误判**：二方公式是配置定义（快照），#67 是核算结果（终值）——**两处对不上时以核算结果为准**，标注配置已改；OpenAPI 拿不到公式时不要凭 #67 数字猜公式

> 二方接口规格见 `api-handbook/03-Gateway接口/薪资项目-列表含公式.md` 与 `薪资项目-公式解析.md`。
