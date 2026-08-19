# 考勤日月报 · API 接口参考与字段含义

> **★ 本文件用途：业务链路速查**——按"问题/模块"找接口链（如"假期额度 → #79→#80→#82→#116"）。
> **要找"某接口编号是什么/在哪个文件"→ 去 `api-handbook/index.md`（接口目录）**；两者定位不同，各管一头。

本文件汇总考勤报表相关的 OpenAPI 接口规格与响应字段含义，配套 `field-formula-source.md`（公式来源规则）与 `attendance-field-logic.md`（计算逻辑）。考勤相关接口全量见 `api-handbook/07-考勤管理/`；跨模块审计接口 #OL 见 `api-handbook/09-操作日志/查询操作日志.md`（手册编号 #110）。

> 字段含义（"是什么"）在此；字段"怎么算"见 `attendance-field-logic.md`；公式"去哪取"见 `field-formula-source.md`。
> **二方接口（#111~#117）**：Gateway 接口（v5.ihr360.com），宿主环境（登录态）实测可用 #116/#117（额度账本类）；其余 #111~#115 不可用或路径待确认。**作为 OpenAPI 的补充**（拿额度流水/锁定账本），非主路径。完整手册见 `api-handbook/03-Gateway接口/`。

---

## 一、接口总览

| 编号 | 接口 | 路径 | 方法 | 用途 |
|------|------|------|------|------|
| #91 | 获取打卡记录 | `/openapi/thirdparty/api/tm/v2/signs/records/search` | POST | 原始打卡时间/地点/来源，验算"实际出勤小时" |
| #93 | 获取日报V4 | `/openapi/thirdparty/api/tm/v4/daily/reports/search` | POST | 日报主数据：标准字段 + 自定义字段值 + 班段签到 |
| #94 | 获取月报V3 | `/openapi/thirdparty/api/tm/v3/period/monthly/reports/search` | POST | 月报主数据：含天数聚合 + 加班拆分 + 两套自定义字段 |
| #96 | 获取日报自定义字段 | `/openapi/thirdparty/api/tm/v1/custom/field/list/search` | GET | 日报自定义字段定义（id→名称/`formula`） |
| #100 | 获取日报自定义字段值 | `/openapi/thirdparty/api/tm/v1/daily/custom/search` | POST | 日报自定义字段实际值（平铺，上限 100人×31天） |
| #101 | 获取月报出勤班次 | `/openapi/thirdparty/api/tm/v1/period/monthly/shift/search` | POST | 月内各班次出勤天数汇总（`shiftAbbrMap`） |
| #102 | 获取月报自定义字段 | `/openapi/thirdparty/api/tm/v1/period/custom/field/list/search` | GET | 月报自定义字段定义（id→名称/`formula`，变量带 `ATT_PERIOD_BASIC$` 前缀） |
| #79 | 获取所有假期类型v2 | `/openapi/thirdparty/api/tm/v2/vacations/types/search` | GET | 假期类型规则配置（单位/折算/最小单位/适用性别/启用） |
| #80 | 获取员工假期余额 | `/openapi/thirdparty/api/tm/v1/vacations/limit/search` | POST | 全部假期当前可用余额（仅 `availableLimit`） |
| #82 | 获取年假余额 | `/openapi/thirdparty/api/tm/v1/vacations/annual/limit/search` | POST | 年假 6 字段明细（法定/福利 × 当年发放/结转/可用） |
| #OL | 查询操作日志 | `/openapi/thirdparty/api/operationlogs/v1/api/query/ids` | POST | 审计日志（跨模块）：查某模块操作 id+subjectName，支撑 B 类"打卡/数据被改"溯源 |
| #111 | 假期设置-限制规则类型 | `/web/gateway/attendance/api/leaveSetting/restriction/types` | GET | 二方：假期设置页的限制规则类型（按 vacationId） |
| #112 | 报表-额度显示类型 | `/web/gateway/attendance/api/limitutil/leaveSettingType` | POST | 二方：假期余报表显示哪些类型（13种 LIMIT_DETAIL） |
| #113 | 报表-额度单假期类型 | `/web/gateway/attendance/api/limitBill/leaveSettingType` | GET | 二方：哪些类型有额度单 |
| #114 | 报表-额度单列表 | `/web/gateway/attendance/api/limitBill/list` | GET | 二方：额度单列表（含 id/leave_setting_id） |
| #115 | 报表-使用记录列表 | `/web/gateway/attendance/api/record/list` | GET | 二方：使用记录列表（含 leave_setting_id） |
| #116 | **报表-额度变更记录** | `/web/gateway/attendance/api/limitBill/record` | GET | **二方：额度变更流水（发放/结转/调整），非年假发放额度唯一来源** |

通用：OpenAPI 鉴权 `Authorization: Bearer {token}`（OAuth client_credentials，AppID:AppSecret 换取，见 SKILL.md「首次使用：API 授权」）；二方鉴权 Cookie+XSRF（宿主环境自动带）；响应 `code=0` 成功；分页 `page` 从 0 起。
> **⚠️ 使用前提**：先完成「首次使用：API 授权」（向客户索取 AppID/AppSecret 换 Bearer），OpenAPI 接口（#79~#102、#OL 等）方可调用。未授权时不得调用 OpenAPI，用现有可用数据降级回答。

---

## 二、接口详细规格

### #93 获取日报V4
- **请求**：`staffId`(否) / `startDate`(是,yyyy-MM-dd) / `endDate`(是) / `page`(默认0) / `size`(默认100,最大1000)
- **响应 content 分组**：
  - 基础：`id` `staffId` `shiftId` `originalShiftId` `shiftName` `signDate` `signType`(正常/异常) `signTypeExplain` `departmentName` `updateDate`
  - 出勤：`supposedAttendanceHours` `actualAttendanceHours` `lateTimes` `lateMinutes` `earlyTimes` `earlyMinutes` `absenceTimes` `absenceHours` `absenceMinutes` `signInMissingTimes` `signOutMissingTimes` `appealTimes`
  - 加班：`normalHours` `weekendHours` `statutoryHours` `deepNightHours` `delayWorkdayHours` `delayDayOffHours` `delayHolidayHours`
  - 出差外出：`businessTravelDays` `fieldWorkHours`
  - 假期：`annualLeave` `personalLeave` `fullPaySickLeave` `paidSickLeave` `marriageLeave` `maternityLeave` `homeLeave` `prenatalCheckup` `paternityLeave` `breastfeedingLeave` `compensatedLeave` `mourningLeave` `parentalLeave` `otherVacation` `customizedLeave`(Map<leaveTypeId,Double>)
  - 班段：`signBlockList[]`(blockNo 1~5；signInTime/Place/Conclusion；signOutTime/Place/Conclusion)
  - 自定义：`dailyCustomField`(Map<字段ID,值>)；`isLock` `isLockName`

### #94 获取月报V3
- **请求**：`staffIds`(否,≤100人) / `periodSettingId`(是,来自#95) / `periodMonth`(是,yyyy-MM) / `page` / `size`(默认100,最大100)
- **响应 content 分组**（与日报差异见下表）：
  - 基础：`id` `staffId` `remark` `updateDate`
  - 出勤（含天数聚合）：`supposedAttendanceDays` `supposedAttendanceHours` `actualAttendanceDays` `actualAttendanceHours` `restDays` `absenceHours` `absenceNumber` `absenceTimes` `absenceMinutes` `lateMinutes` `lateTimes` `earlyMinutes` `earlyTimes` `signInMissingTimes` `signOutMissingTimes` `appealTimes`
  - 出差外出：`businessTravelDays` `fieldWorkHours`
  - 加班（拆分 总量/转调休/转薪资）：`normalHour` `weekendHour` `statutoryHour` `normalToRestHour` `weekendToRestHour` `statutoryToRestHour` `normalToSalaryHour` `weekendToSalaryHour` `statutoryToSalaryHour` `workdayDelayHours` `dayOffDelayHours` `holidayDelayHours` `deepNightDuration`
  - 假期：同日报 15 类（含 `customizedLeave`）
  - 自定义：`periodCustomField`(日报汇总) `periodCustomFieldInfo`(月报专属)；`isLock` `isLockName`

**#93 vs #94 关键差异**

| 维度 | 日报 #93 | 月报 #94 |
|------|---------|---------|
| 时间参数 | startDate/endDate | periodSettingId + periodMonth |
| 人员 | staffId（单人） | staffIds[]（≤100） |
| 天数统计 | 无 | supposedAttendanceDays / actualAttendanceDays / restDays |
| 加班 | 仅总时长（7项） | Total+ToRest+ToSalary（13项） |
| 班段签到 | signBlockList[1~5] | 无 |
| 自定义字段 | dailyCustomField | periodCustomField + periodCustomFieldInfo |
| 分页上限 | 1000 | 100 |

### #93 vs #96 关联（★必须，2026-08-13 宿主实测补）

**#93 返回的 `dailyCustomField` 是「字段UUID → 值」映射，必须调用 #96 拿到字段定义（id→名称/formula），才能展示可读的字段名和计算逻辑**——否则只看到一串 UUID，无法回答"这个字段是什么/怎么算"。

**查询链路**：
```
#93 日报 → dailyCustomField（UUID→值）→ 提取 UUID 列表
→ #96 日报自定义字段定义（id→名称/formula）
→ 按 id 关联 → 输出"字段名称 → 公式逻辑 → 当日值"
```

**输出要求**：回答日报自定义字段问题时，**必须同时查 #96**；展示时列出 字段名称 / 公式逻辑 / 当日值；公式引用语法见 #96 段（`出勤$字段名` / `日报自定义字段$字段名` / `员工基本信息$字段名` 等）。

### #96 获取日报自定义字段（GET，无参）
- **响应 data[]**：`id` `companyId` `name` `valueType`(NUMBER/TEXT) `unit` `roundingMode` `roundingNumber` `numberSetting` `greaterOrEqualNumber` `lessOrEqualNumber` `isRawData` `effectiveDate` `statisticsMonthlyReport` `enabled` `remark` **`formula`**
- **formula 示例**：`=IF(hour(出勤$班段1签到时间)>=11,1,0)`、`=员工基本信息$姓名`
- 关联：本接口 `id` → #93 `dailyCustomField` 的 key、#94 `periodCustomField` 的 key

### #102 获取月报自定义字段（GET，无参）
- 同 #96 结构，差异：
  - 生效维度：`month`(yyyy-MM)，无 `effectiveDate`、`statisticsMonthlyReport`（恒 null）
  - **formula 变量带 `ATT_PERIOD_BASIC$` 前缀**，如 `=ATT_PERIOD_BASIC$supposedAttendanceHours`
- 关联：本接口 `id` → #94 `periodCustomFieldInfo` 的 key

### #100 获取日报自定义字段值（POST）
- 路径 `/openapi/thirdparty/api/tm/v1/daily/custom/search`，返回平铺的 `dailyCustomField`，与 #96 的 `id` 对应；上限 100人×31天。是 #93 `dailyCustomField` 的独立可单独拉取版本。

### #91 获取打卡记录（POST）
- **请求**：`startTime`(是) / `endTime`(是,跨度≤31天) / `staffIdList`(否) / `page` / `size`(最大1000)
- **响应 content[]**：`staffId` `signTime`(时间戳) `signAddress` `source`(来源大类) `sourceDescription`(具体客户端) `isActived`
- `source` 枚举（20种）：MACHINE 考勤机 / NORMAL 正常 / APPEAL 补卡 / TEAM_SIGN 团队打卡 / OUT_SIGN 外勤 / HR_OPERATION / HR_JUDGE / OLD_RECORD / OVERTIME 加班 / HR_APPEAL / HR_APPEAL_IMPORT / APP / WECHAT / DINGDING / OPEN_API / FEISHU / FANWEI 泛微 / YUNZHIJIA 云之家 / CIMOS / WPSOA

### #101 获取月报出勤班次（POST）
- **请求**：`periodSettingId`(是) / `periodMonth`(是,yyyy-MM) / `page` / `size`(≤100)
- **响应 content[]**：`staffId` `departmentId` `departmentName` `shiftAbbrMap`(Map<班次简称,出勤天数>)
- 用途：统计员工当月每种班次上了几天（计次非计小时），对应月报"出勤班次"字段。

### #OL 查询操作日志（POST）— 审计/溯源
- 已独立归档为手册编号 **#110**，完整规格见 `api-handbook/09-操作日志/查询操作日志.md`。
- **路径**：`/openapi/thirdparty/api/operationlogs/v1/api/query/ids`
- **请求 body**：`companyId`(**是**) / `page`(是,从1起) / `size`(是,≤100) / `sort`(如["operationTime"]) / `operationModule`(是) / `operationType`(否) / `subjectName`(否) / `operationTimeRangeStart`(是) / `operationTimeRangeEnd`(是)
- **响应 data**（实测为**对象**，非官方文档误标的 Boolean）：`totalPages` `totalElements` `end` `content[]`，其中 content 每项 = `{id, subjectName}`
- **operationModule 枚举**（部分）：`ATTENDANCE_SIGN` 考勤打卡 / `TASK_MANAGEMENT` 任务管理
- **operationType 枚举**：`VIEW` 查询 / `INSERT` 新增 / `UPDATE` 编辑 / `DELETE` 删除 / `IMPORT` 导入 / `EXPORT` 导出 / `OPEN_API` OpenAPI接入 / `BUSINESS_ARCHIVE` 业务归档
- **subjectName 实测样例**：`流程审批补卡单通过` / `HR新增考勤了打卡记录` / `通过OpenApi删除了打卡记录`
- **用途（B 类异常诊断）**：排查"实际出勤/打卡为何对不上"时，先用本接口查时间窗内 `ATTENDANCE_SIGN` 是否有 `DELETE`/`OPEN_API`/`INSERT` 操作，确认打卡记录是否被人为改动；`subjectName` 为通用描述，**不含员工姓名**，需结合 #91 打卡记录交叉定位具体员工。
- **注意**：官方文档响应参数表将 `data` 类型误写为 Boolean（"是否更新成功"），实际返回分页对象，以实测示例为准。

### #79 获取所有假期类型v2（GET，无参）
- **响应 data[]**（21 种，字段极多，重点用以下业务字段）：
  - `id` 假期类型ID / `leaveName` 假期名 / `vacationType` 类型枚举（ANNUAL_LEAVE 年假 / ADJUST_REST 调休 / AFFAIR_LEAVE 事假 / FUNERAL_LEAVE 丧假 / HOME_LEAVE 探亲假 / LACTATION_LEAVE 哺乳假 / MARITAL_LEAVE 婚假 / MATERNITY_LEAVE 产假 / PARENTAL_LEAVE 育儿假 / PATERNITY_LEAVE 陪产假 / PRENATAL_CHECK_UP 产检假 / SABBATICAL_LEAVE 公休 / SICK_LEAVE 病假 / USER_DEFINED_VACATION 自定义假期 / OTHER_VACATION 其他假期）
  - `standardUnit`/`calculationUnit` 计发单位：DAY 天 / HOUR 小时（病假、哺乳假、季度假、自定义调休等按小时）
  - `smallestUnit` 最小申请单位：**DAY 型=天数（0.5 半天、1 一天）；HOUR 型=分钟数（实测：哺乳假60/病假120/季度假30/自动发额度小时假1）**
  - `stepUnit` 申请步长（有值才限制，如事假 0.5 天）
  - `maximumUnit` 单次最大申请量（事假 3）
  - `calculationHours` / `dayHourConversion` 1天折算小时数（有值才显示"1天=X小时"，如事假 8）
  - `sexScope` 适用性别：FEMALE 仅女性（哺乳/产/产检假）/ MALE 仅男性（陪产假）/ UNLIMITED 或 null 不限
  - `using` 是否启用（false = 客户侧不可申请，如"其他假期"）
- **用途**：① 把 #80 返回的 `vacationId` 映射成中文假期名；② 展示"假期规则"（单位/折算/最小单位/适用性别）。

### #80 获取员工假期余额（POST）
- **请求**：`staffId`(是) / `vacationIdList`(是,List<假期类型ID>)
- **响应 data[]**：每条仅 `{vacationId, availableLimit}` 两个字段——**所有假期类型都只有一个"当前可用余额"数字，无任何计算过程/明细**（官方文档 + 实测一致确认）
- **关键边界（问题2 定案依据）**：非年假假期的"发放额度来源" API 拿不到；要拼"余额=发放−已用+结转"需另调 #77 休假单（已用）与 #86 结转（转余额）
- **实测 edge case**：公休 availableLimit 可为负（-61.0），表示超用额度，展示时需特殊处理
- **变更历史**：2021-08-19 创建（Kayle Pan）；2023-12-28 修改假期可用余额参数为 double（Jiabao Lu）
- **请求示例**：`{"staffId":"...","vacationIdList":["46ed3891-..."]}`
- **响应示例**：`{"code":0,"message":"OK","data":[{"vacationId":"abb6c0dc-...","availableLimit":1.0},{"vacationId":"46ed3891-...","availableLimit":1.0}],"errorResult":false}`

### #82 获取年假余额（POST）
- **请求**：`staffId`(是)
- **响应 data**（6 字段，**唯一有明细拆分的假期**）：
  - 法定年假：`annualAvailableLimit` 可用 / `annualAvailableIssueLimit` 当年发放 / `annualCarryAvailableLimit` 结转
  - 福利年假：`welfareAvailableLimit` 可用 / `welfareAvailableIssueLimit` 当年发放 / `welfareCarryAvailableLimit` 结转
- 实测样例：法定 当年5 / 结转5 / 可用5；福利 全部 0

### #111~#115 二方接口（Gateway，摘要）
- **鉴权**：Cookie + XSRF（v5.ihr360.com，宿主内置 AI 环境自动携带），无需 Bearer token
- **#111 假期设置-限制规则类型**：`GET /web/gateway/attendance/api/leaveSetting/restriction/types?vacationId={id}` 返回某假期的规则配置（限制类型）
- **#112 报表-额度显示类型**：`POST /web/gateway/attendance/api/limitutil/leaveSettingType` 返回报表显示哪些假期类型（实测 13 种，均 `showLimitType=LIMIT_DETAIL`）
- **#113 报表-额度单假期类型**：`GET /web/gateway/attendance/api/limitBill/leaveSettingType` 返回有额度单的假期类型
- **#114 报表-额度单列表**：`GET /web/gateway/attendance/api/limitBill/list?id={}&leave_setting_id={}` 额度单列表（发放/调整单）
- **#115 报表-使用记录列表**：`GET /web/gateway/attendance/api/record/list?leave_setting_id={}` 使用记录
- 完整规格：`api-handbook/03-Gateway接口/`（假期余报表-额度显示类型.md / 额度单假期类型.md / 额度单列表.md / 使用记录列表.md）

### #116 报表-额度变更记录（二方，★核心★）
- **端点**：`GET /web/gateway/attendance/api/limitBill/record`
- **参数**：`staffId`(是) / `vacationId`(是,假期类型ID) / `page`(否,从0) / `size`(否) —— **r_id/u_id 实测后端不校验，可省略**
- **用途**：返回某员工某假期的**额度变更流水**（发放/结转/调整/作废），**非年假假期发放额度的唯一来源**
- **响应 data.content[]** 每项：
  - `operateTime` 操作时间 / `operateReason` **操作原因（自然语言，含额度来源说明）** / `operateLimit` **操作额度（带符号+单位，如 "+5.0天"、"-8.0小时"）** / `expirationTime` 有效期(毫秒) / `carryoverTime` 结转时间 / `dataId` 关联单据ID
- **实测样例**（年假）：`周期开始计算（当前周期法定年假增加5.0天） +5.0天` / `上一周期剩余法定年假额度结转到本周期，有效期：2026-08-19（上周期结转法定年假增加5.0天） +5.0天` / `法定年假结转到期日作废 -4.15天`
- **实测样例**（调休，额度来源可溯源）：`加班单据初始化导入重算修改 +20.0小时` / `删除加班单 -20.0小时` / `员工申请调休 -8.0小时` / `HR通过调休表调整 -0.25小时` / `作废加班单 -2.0小时`
- **与其他接口配合**：额度流水（#116 发放/结转）+ 休假单（#77 已用）+ 余额（#80/#82）→ **完整计算链还原**（发放−已用+结转−作废=余额）
- **批量查询**：员工清单取 OpenAPI #1（在职）+ 假期类型取 #79 → 双层循环拉全量；r_id 不校验免维护

---

## 三、字段含义对照表

### 日报响应字段（#93）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| id | 日报 ID | String | |
| staffId | 员工 ID | String | |
| shiftId / originalShiftId | 班次 ID / 原始班次 ID | String | |
| shiftName | 班次名称 | String | 如"朝九晚五" |
| signDate | 日期 | Date | |
| signType / signTypeExplain | 状态(正常/异常) / 状态说明 | String | 异常时含"缺勤"等 |
| departmentName | 部门名称 | String | |
| updateDate | 更新时间 | Date | yyyy-MM-dd HH:mm:ss |
| supposedAttendanceHours | 应出勤小时数 | Double | **班次决定** |
| actualAttendanceHours | 实际出勤小时数 | Double | 打卡∩班次−休息，封顶 |
| lateTimes / lateMinutes | 迟到次数 / 迟到分钟数 | Integer | |
| earlyTimes / earlyMinutes | 早退次数 / 早退分钟数 | Integer | 开放二次加工系统字段 |
| absenceTimes / absenceHours / absenceMinutes | 缺勤次数 / 缺勤小时数 / 缺勤分钟数 | Integer/Double | |
| signInMissingTimes / signOutMissingTimes | 签到缺卡 / 签退缺卡 次数 | Integer | |
| appealTimes | 补卡次数 | Integer | |
| normalHours / weekendHours / statutoryHours | 工作日/休息日/节假日 加班时长 | Double | |
| deepNightHours | 深夜工时 | Double | |
| delayWorkdayHours / delayDayOffHours / delayHolidayHours | 工作日/休息日/节假日 延时工时 | Double | |
| businessTravelDays | 出差（天） | Double | |
| fieldWorkHours | 外出（小时） | Double | |
| annualLeave … otherVacation | 年假/事假/全薪病假/扣薪病假/婚假/产假/探亲/产检/陪产/哺乳/调休/丧假/育儿/其他假期 | Double | 单位见假期类型 calculationUnit |
| customizedLeave | 自定义假期 | Map<String,Double> | key=leaveTypeId（#79） |
| signBlockList | 班段签到信息 | JSONArray | blockNo 1~5，含签到/签退时间·地点·结论 |
| dailyCustomField | 日报自定义字段值 | Map<String,String> | key=自定义字段 ID（#96） |
| isLock / isLockName | 是否锁定 / 锁定名称 | Boolean/String | 2025-06-24 新增 |

### 月报响应字段（#94，含月报特有）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| supposedAttendanceDays | 应出勤天数 | Double | **月报特有** |
| actualAttendanceDays | 实际出勤天数 | Double | 月报特有 = 实际出勤小时÷应出勤小时，最大取1 |
| restDays | 排休天数 | Double | 月报特有 |
| absenceNumber | 缺勤天数 | Double | 月报特有 |
| normalHour / weekendHour / statutoryHour | 工作日/休息日/节假日 加班总时长 | Double | 月报拆分 |
| normalToRestHour … statutoryToSalaryHour | 对应 转调休 / 转薪资 | Double | 9 项拆分 |
| workdayDelayHours / dayOffDelayHours / holidayDelayHours | 工作日/休息日/节假日 延时工时 | Double | |
| deepNightDuration | 深夜工时 | Double | 月报命名 |
| periodCustomField | 日报汇总到月报的自定义字段 | Map<String,String> | key=日报自定义字段 ID |
| periodCustomFieldInfo | 月报自定义字段 | Map<String,String> | key=月报自定义字段 ID（#102） |

> 月报其余出勤/异常/加班总时长/假期字段与日报同名同义，不重复列。

### 自定义字段定义（#96 / #102）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| id | 自定义字段 ID | String | 与值接口 Map 的 key 对应 |
| name | 字段名称 | String | |
| valueType | 字段类型 | String | NUMBER / TEXT |
| unit | 字段单位 | String | |
| roundingMode | 圆整方式 | String | 8 种（见下） |
| roundingNumber | 圆整基数 | Double | |
| numberSetting | 数字设置 | String | GREATER_OR_EQUAL / LESS_OR_EQUAL / BETWEEN |
| isRawData | 被引用时用原始数据 | Boolean | |
| effectiveDate / month | 生效日期 / 生效月份 | Date/String | 日报用日期，月报用 yyyy-MM |
| statisticsMonthlyReport | 是否统计到月报 | Boolean | 日报特有 |
| enabled | 是否启用 | Boolean | |
| formula | **自定义公式** | String | 类 Excel；取"该字段怎么算"的唯一明文来源 |

**枚举速查**
- `valueType`：NUMBER 数值 / TEXT 文本
- `roundingMode`：ROUND_UP 向上 / ROUND_DOWN 向下 / AN_INTEGER 舍小数 / CARRY_INTEGER 进位取整 / ROUND_KEEP_TWO 四舍五入留小数 / DISCARD_KEEP_TWO 舍尾留小数 / ROUND 四舍五入取整 / CARRY_DECIMALS 进位留小数
- `numberSetting`：GREATER_OR_EQUAL / LESS_OR_EQUAL / BETWEEN

### 打卡记录字段（#91）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| staffId | 员工 ID | String | |
| signTime | 打卡时间 | Date(时间戳) | |
| signAddress | 打卡地点 | String | |
| source | 来源大类 | String | 20 种枚举 |
| sourceDescription | 具体客户端 | String | 如 APP |
| isActived | 是否有效 | Boolean | |

### 月报出勤班次（#101）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| staffId | 员工 ID | String | |
| departmentId / departmentName | 部门 ID / 名称 | Long/String | |
| shiftAbbrMap | 班次出勤天数 | Map<String,Integer> | key=班次简称，value=出勤天数 |

### 操作日志（#OL）

| 字段(英文) | 中文含义 | 类型 | 备注 |
|-----------|---------|------|------|
| id | 操作日志 ID | String | 数值型字符串（如 1133964610173665280） |
| subjectName | 操作目标名称 | String | 通用描述，不含员工姓名；如"HR新增考勤了打卡记录" |

> 请求侧关键入参：`companyId`(必填，租户ID) / `operationModule`(必填，如 ATTENDANCE_SIGN) / `operationTimeRangeStart~End`(必填，ISO8601 含时区，如 2026-07-25T00:00:00+08:00) / `operationType`(可选过滤)。

---

## 四、公式变量说明

- **日报公式变量**：引用打卡/班次变量，如 `出勤$班段1签到时间`、`员工基本信息$姓名`
- **月报公式变量**：统一带 `ATT_PERIOD_BASIC$` 前缀，如 `ATT_PERIOD_BASIC$supposedAttendanceHours`
- 公式语法为类 Excel（IF / hour / 算术运算等），由系统解析执行；企业可改写，API 返回的是改写后的明文。
