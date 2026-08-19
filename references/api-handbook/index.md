# iHR360 API 参考手册

> **★ 本文件用途：接口目录 / 编号索引**——"某接口编号是什么、在哪个文件"在这里查。
> **要找"某业务问题调哪些接口"→ 去 `references/api-reference.md`（业务链路速查）**；两者定位不同，各管一头。
>
> 来源：OpenAPI 官方文档 + Gateway 实际验证  
> 生成日期：2026-06-17  
> 总接口数：110 个

---

## 鉴权说明

| 类型 | 说明 | 适用接口 |
|------|------|---------|
| **OAuth Bearer** | OpenAPI，OAuth client_credentials 换取：POST `/openapi/oauth/token`（Basic base64(AppID:AppSecret)，body `grant_type=client_credentials&scope=client`），有效期 2h | `openapi.ihr360.com` 路径 |
| **IHR360_API_TOKEN（宿主 AI 环境）** | iHR 内置 AI 宿主自动注入环境变量；调用时 `Authorization: Bearer {IHR360_API_TOKEN}`，Base URL `https://v5.ihr360.com`，路径前缀 `/web/gateway/...` | 二方 Gateway 接口（#5~#8 + #111~#117） |
| **Cookie+XSRF** | 前端浏览器插件自动携带 | 前端 HTML 插件调用 Gateway 接口 |

> ⚠️ **使用 OpenAPI 前必须先完成授权**：向客户索取开放平台 AppID/AppSecret（`openapi.ihr360.com` → 应用管理），走 OAuth 换取 Bearer token——完整步骤见 `SKILL.md`「首次使用：API 授权」小节。未授权不得调用 OpenAPI。
>
> ⚠️ **使用二方 Gateway 接口前必须先检查环境变量**：进入会话后确认 `IHR360_API_TOKEN` 是否存在，存在则二方接口直接可用——详细调用方式见 `SKILL.md`「宿主登录态」小节。
>
> ⚠️ **OpenAPI OAuth Bearer 与 IHR360_API_TOKEN 不互通**：OpenAPI 的 token 调 `v5.ihr360.com` 会 403；二方接口的 token 调 `openapi.ihr360.com` 也会 401。

---

## 模块索引

### [01-组织架构](./01-组织架构/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [部门](./01-组织架构/部门.md) | #9, #13, #14 | 部门搜索/详情/元数据 |
| [职位](./01-组织架构/职位.md) | #15, #16, #17, #18 | 职位搜索/按部门查/按ID查/职级 |
| [职务·职级·体系表](./01-组织架构/职务-职级-体系表.md) | #19, #20, #21 | 职务清单/职级清单/体系表 |
| [法律实体·工作地点·职务分类](./01-组织架构/法律实体-工作地点-职务分类.md) | #22, #23, #24 | 法律实体/工作地点/职务分类 |
| [多维组织](./01-组织架构/多维组织.md) | #25 | 多维组织查询 |

### [02-员工管理](./02-员工管理/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [员工基础信息](./02-员工管理/员工基础信息.md) | #1, #2, #3, #26, #27, **手机号反查(无编号)** | ID清单/详情/直属领导/删除/复职/**手机号→staffId** |
| [员工合同](./02-员工管理/员工合同.md) | #4, #35 | 合同清单/当前合同 |
| [员工信息子集](./02-员工管理/员工信息子集.md) | #28, #29, #30, #31, #32, #33, #34, #36, #37 | 任职/教育/工作/证照/银行卡/上级/兼任/自定义子集 |
| [多维组织任职记录](./02-员工管理/多维组织任职记录.md) | #52 | 多维组织任职 |
| [入职](./02-员工管理/入职.md) | #38, #39, #40 | 字段定义/待入职清单/详情 |
| [转正](./02-员工管理/转正.md) | #41, #42, #43 | 按日期查/V2当天/按ID批量 |
| [调动](./02-员工管理/调动.md) | #44, #45, #46 | 待调动/时间段/设置 |
| [离职](./02-员工管理/离职.md) | #47, #48, #49 | 待离职/当日/时间段 |
| [绩效档案](./02-员工管理/绩效档案.md) | #50 | 绩效档案查询 |
| [家庭成员](./02-员工管理/家庭成员.md) | #51 | 家庭成员查询 |

### [03-Gateway接口](./03-Gateway接口/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [Gateway接口](./03-Gateway接口/Gateway接口.md) | #5, #6, #7, #8 | 花名册搜索/详情/离职/年假 |
| [假期设置-限制规则](./03-Gateway接口/假期设置-限制规则.md) | #111 | 假期设置-限制规则类型（二方） |
| [假期余报表-额度显示类型](./03-Gateway接口/假期余报表-额度显示类型.md) | #112 | 额度显示类型（二方，POST，13种） |
| [假期余报表-额度单假期类型](./03-Gateway接口/假期余报表-额度单假期类型.md) | #113 | 额度单假期类型（二方，GET） |
| [假期余报表-额度单列表](./03-Gateway接口/假期余报表-额度单列表.md) | #114 | 额度单列表（二方，GET） |
| [假期余报表-使用记录列表](./03-Gateway接口/假期余报表-使用记录列表.md) | #115 | 使用记录列表（二方，GET） |
| [假期余报表-额度变更记录](./03-Gateway接口/假期余报表-额度变更记录.md) | #116 | **额度变更记录（二方，✅实测，问题2核心）** |
| [累计转薪资结余记录](./03-Gateway接口/累计转薪资结余记录.md) | #117 | **转薪资结余记录（二方，含锁定账本，问题3核心）** |
| [薪资项目-列表含公式](./03-Gateway接口/薪资项目-列表含公式.md) | — | **薪资项目全量 185 项含 formula 明文（问题4公式来源）** |
| [薪资项目-公式解析](./03-Gateway接口/薪资项目-公式解析.md) | — | **薪资公式 exprItems token 流解析（问题4核心）** |

### [04-枚举值速查](../枚举值速查.md)

所有接口的枚举值完整汇总。

### [05-薪资管理](./05-薪资管理/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [获取公司薪资方案列表](./05-薪资管理/薪资方案.md) | #55 | 全公司薪资方案列表 |
| [获取公司薪资方案详情](./05-薪资管理/薪资方案.md) | #56 | 按 planId 查详情（含员工+字段） |
| [基于方案ID查询核算中薪资核算表V2](./05-薪资管理/基于方案ID查询核算中薪资核算表V2.md) | #57 | 按方案+年月查核算中薪资数据（v2） |
| [薪资核算根据审批id查询审批文件地址](./05-薪资管理/薪资核算根据审批id查询审批文件地址.md) | #58 | 通过审批ID查核算文件地址 |
| [员工薪资调整](./05-薪资管理/员工薪资调整.md) | #59 | PUT 更新员工薪资档案字段（写入） |
| [获取员工薪资档案调整记录](./05-薪资管理/薪资档案与月度结果.md) | #60 | 批量查调整记录（含调整前后值） |
| [获取员工个税扣缴义务人集合](./05-薪资管理/个税扣缴义务人.md) | #61 | 查个税扣缴义务人历史（GET单员工） |
| [获取员工所有个税扣缴义务人和薪资档案](./05-薪资管理/个税扣缴义务人.md) | #62 | 一步到位查个税+薪资当前值（批量POST） |
| [薪资结果定义列表](./05-薪资管理/薪资结果定义.md) | #63 | 查哪些年月有薪资核算结果（入口接口） |
| [薪资结果定义详情](./05-薪资管理/薪资结果定义.md) | #64 | 按方案+年月查详情（员工+字段定义） |
| [基于部门ID和月份查询薪资结果表](./05-薪资管理/查询薪资结果表.md) | #65 | 按部门查员工月度薪资明细 |
| [基于人员MobileNo和月份查询薪资结果表](./05-薪资管理/查询薪资结果表.md) | #66 | 按手机号查员工月度薪资明细 |
| [基于人员ID列表和月份查询薪资结果表](./05-薪资管理/查询薪资结果表.md) | #67 | 按员工ID查员工月度薪资明细 |
| [系统薪资项目列表](./05-薪资管理/薪资项目.md) | #68 | 全系统内置薪资项目（含数据类型） |
| [根据薪资项目获取员工月度薪资结果](./05-薪资管理/薪资档案与月度结果.md) | #69 | 按薪资项目 code 分页查月度结果 |
| [获取应用于员工薪资档案的薪资项目](./05-薪资管理/薪资项目.md) | #70 | 公司已启用的薪资档案项目 |
| [获取公司所有薪资项目](./05-薪资管理/薪资方案.md) | #71 | 公司所有薪资项目 |
| [社保缴纳方案集合](./06-福利管理/社保缴纳方案集合.md) | #72 | 获取社保/公积金/其他福利方案列表 |
| [社保方案获取基数字段](./06-福利管理/社保方案获取基数字段.md) | #73 | 社保方案基数字段 |
| [公积金方案获取基数字段](./06-福利管理/公积金方案获取基数字段.md) | #74 | 公积金方案基数字段 |
| [其他福利方案获取基数字段](./06-福利管理/其他福利方案获取基数字段.md) | #75 | 其他福利方案基数字段 |
| [基于年月和人员ID查询已存档社保台账结果](./06-福利管理/基于年月和人员ID查询已存档社保台账结果.md) | #76 | 按年月+员工查社保台账明细 |

### [07-考勤管理](./07-考勤管理/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [获取休假单](./07-考勤管理/休假与节假日.md) | #77 | 分页查询公司员工休假单列表 |
| [获取假期类型v1](./07-考勤管理/假期.md) | #78 | 假期类型列表（v1，基础信息） |
| [获取所有假期类型v2](./07-考勤管理/假期.md) | #79 | 假期类型详情（v2，含额度规则） |
| [获取一个员工假期余额](./07-考勤管理/假期.md) | #80 | 查员工特定假期类型的可用余额 |
| [获取法定节假日](./07-考勤管理/休假与节假日.md) | #81 | 查法定节假日放假/补班日期 |
| [获取年假余额](./07-考勤管理/假期.md) | #82 | 查员工法定+福利年假余额 |
| [获取加班单v2](./07-考勤管理/加班.md) | #83 | 查已生效加班单列表（v2） |
| [获取加班设置](./07-考勤管理/加班.md) | #84 | 查员工加班规则配置 |
| [加班单时长试算](./07-考勤管理/加班.md) | #85 | 试算加班时长 |
| [根据结转日期获取加班单](./07-考勤管理/加班.md) | #86 | 按结转日期查加班单 |
| [获取外出单V2](./07-考勤管理/获取外出单V2.md) | #87 | 查外出单列表 |
| [获取出差单V2](./07-考勤管理/获取出差单V2.md) | #88 | 查出差单列表 |
| [获取补卡原因](./07-考勤管理/获取补卡原因.md) | #89 | 按员工ID查补卡原因列表 |
| [获取补卡单](./07-考勤管理/获取补卡单.md) | #90 | 按时间分页查补卡单列表 |
| [获取打卡记录](./07-考勤管理/获取打卡记录.md) | #91 | 按时间段查打卡记录（20种来源类型） |
| [获取排班V4](./07-考勤管理/获取排班V4.md) | #92 | 批量查排班班次（v4，≤100人） |
| [获取日报V4](./07-考勤管理/日报.md) | #93 | 日报数据V4（40+字段） |
| [获取月报V3](./07-考勤管理/月报.md) | #94 | 月报数据V3（50+字段） |
| [获取考勤周期设置](./07-考勤管理/考勤周期.md) | #95 | 所有考勤周期设置列表 |
| [获取日报自定义字段](./07-考勤管理/日报.md) | #96 | 日报自定义字段定义 |
| [获取员工考勤周期设置](./07-考勤管理/考勤周期.md) | #97 | 按员工ID查考勤周期归属 |
| [根据考勤周期id获取员工设置](./07-考勤管理/考勤周期.md) | #98 | 按周期ID查员工归属 |
| [获取考勤周期实例](./07-考勤管理/考勤周期.md) | #99 | 周期实例运行时状态 |
| [获取日报自定义字段值](./07-考勤管理/日报.md) | #100 | 查日报自定义字段实际值 |
| [获取月报出勤班次](./07-考勤管理/月报.md) | #101 | 按考勤周期汇总各班次出勤次数 |
| [获取月报自定义字段](./07-考勤管理/月报.md) | #102 | 月报自定义字段定义 |

### 08-审批相关

| 文件 | 编号 | 说明 |
|------|------|------|
| [获取第三方选项数据GET](./08-审批相关/获取第三方选项数据GET.md) | #103 | OA回调：GET第三方选项 |
| [获取第三方选项数据POST](./08-审批相关/获取第三方选项数据POST.md) | #104 | OA回调：POST第三方选项（动态过滤） |
| [通过流程id和审批人id获取待办](./08-审批相关/通过流程id和审批人id获取待办.md) | #105 | 查待办链接 |
| [根据参数查询审批已完成的单据数据](./08-审批相关/根据参数查询审批已完成的单据数据.md) | #106 | 多维筛选已完结单据 |
| [根据流程id修改审批流程状态](./08-审批相关/根据流程id修改审批流程状态.md) | #107 | 修改三方推送审批状态 |
| [根据标记码变更流程实例数据](./08-审批相关/根据标记码变更流程实例数据.md) | #108 | 按标记码更新表单（需JWT） |
| [通过流程ID审批节点审批人审批状态执行审批任务](./08-审批相关/通过流程ID审批节点审批人审批状态执行审批任务.md) | #109 | 模拟执行审批任务 |
| [审批类型分组-列表](./08-审批相关/审批类型分组-列表.md) | 二方 | ★审批类型分组列表（问题5 定位第1层） |
| [审批设置-分组及类型列表](./08-审批相关/审批设置-分组及类型列表.md) | 二方 | ★审批分组+类型全量（含 approvalSettingId/processUniqueId/formId 三件套） |
| [审批流权限分组-员工部门范围](./08-审批相关/审批流权限分组-员工部门范围.md) | OpenAPI | ★员工分组查询（员工→适配流程，2026-08-10 实测） |
| [审批类型设置-流程定义](./08-审批相关/审批类型设置-流程定义.md) | 二方 | ★流程定义（条件 Gateway/会签/并行/审批人 + hasUnPublishDraft 解锁） |
| [出差流程整体逻辑-实测案例](./08-审批相关/出差流程整体逻辑-实测案例.md) | 案例 | ★出差全链路实测（表单/单据/source 跨年/鉴权边界） |

### [09-操作日志](./09-操作日志/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [查询操作日志](./09-操作日志/查询操作日志.md) | #110 | 跨模块审计/溯源：按模块+时间窗查操作记录（新增/编辑/删除/OpenAPI 等） |

---


### [10-权限管理](./10-权限管理/README.md)

| 子模块 | 接口编号 | 说明 |
|--------|---------|------|
| [authcenter-用户信息-me](./10-权限管理/authcenter-用户信息-me.md) | 无编号 | 当前用户权限信息（authorities 角色/admin 标志/boundCompanyList 多公司，2026-08-18 抓包） |
| [authority-用户菜单树](./10-权限管理/authority-用户菜单树.md) | 无编号 | 用户功能权限菜单树（menu[]/authorityId，2026-08-18 抓包） |
| [authcenter-简单用户-手机号校验](./10-权限管理/authcenter-简单用户-手机号校验.md) | 无编号 | 授权第1步：手机号是否存在（existByMobile，2026-08-18 抓包） |
| [authcenter-简单用户-更新创建](./10-权限管理/authcenter-简单用户-更新创建.md) | 无编号 | 授权第2步：创建简单用户返回 userId（simpleUser/update） |
| [authcenter-子HR-管理员列表](./10-权限管理/authcenter-子HR-管理员列表.md) | 无编号 | 授权第3步：子HR/管理员列表（simpleUser/subHr/list） |
| [authcenter-简单用户-批量查询](./10-权限管理/authcenter-简单用户-批量查询.md) | 无编号 | 用户详情含角色+数据权限（listByIds → roleList/dataAuths） |
| [authority-角色管理](./10-权限管理/authority-角色管理.md) | 无编号 | 角色CRUD：isNameExist校验/updateRole创建/role/page列表（用例2核心） |
| [manage-公司应用列表](./10-权限管理/manage-公司应用列表.md) | 无编号 | 已开通应用（associatedFunctionIds 功能权限点，2026-08-18 抓包） |
| [manage-合同公司列表](./10-权限管理/manage-合同公司列表.md) | 无编号 | 合同公司（contractCorpIds 数据源 + departmentIds 关联，2026-08-18 抓包） |
| [authority-角色功能树](./10-权限管理/authority-角色功能树.md) | 无编号 | 查角色已分配功能：role_fun/v2/tree?roleId（APP/PC双端+check标记，与authorizationForRole配对，2026-08-18 抓包） |
| [authority-用户绑定角色](./10-权限管理/authority-用户绑定角色.md) | 无编号 | 用户绑定角色 bind_role + whileBind 候选 + 角色功能授权 authorizationForRole（用例1核心，2026-08-18 抓包） |
| [authority-用户数据授权](./10-权限管理/authority-用户数据授权.md) | 无编号 | 数据范围完整版：基础层+功能级authorityTree+数据源（treeSelector/Corporation），2026-08-18 抓包 |

## 快速接口对照表

| # | 接口名称 | 路径 | 方式 | 鉴权 | 模块 |
|---|---------|------|------|------|------|
| 1 | 获取员工ID清单 | `/openapi/thirdparty/api/staff/v1/staffs/ids` | GET | Bearer | 员工管理 |
| 2 | 获取员工信息详情 | `/openapi/thirdparty/api/staff/v1/staffs/{staffId}/detail` | GET | Bearer | 员工管理 |
| 3 | 员工信息详情（含领导） | `/openapi/thirdparty/api/staff/v1/staffs/{staffId}/superiors/detail` | GET | Bearer | 员工管理 |
| 4 | 获取员工合同清单 | `/openapi/thirdparty/api/staff/v1/contracts` | POST | Bearer | 员工管理 |
| 5 | 花名册员工搜索 | `/web/gateway/roster/aggregate/v1/staffs/search` | POST | Cookie+XSRF | Gateway |
| 6 | 员工花名册详情批量 | `/gateway/hr/core/api/employee/detail/list` | POST | Cookie+XSRF | Gateway |
| 7 | 离职清单 | `/gateway/hr/core/api/dimission/list` | POST | Cookie+XSRF | Gateway |
| 8 | 年假报表 | `/gateway/attendance/leave/api/annual/report/summary` | POST | Cookie+XSRF | Gateway |
| 9 | 获取部门清单v3 | `/openapi/thirdparty/api/org/v1/organizations/search` | POST | Bearer | 组织架构 |
| 10 | 批量创建部门 | `/openapi/thirdparty/api/org/v1/organizations/create` | POST | Bearer | 组织架构 |
| 11 | 更新部门 | `/openapi/thirdparty/api/org/v1/organizations/{departmentId}/update` | POST | Bearer | 组织架构 |
| 12 | 设置部门负责人 | `/openapi/thirdparty/api/org/v1/organizations/principal/set` | POST+Query | Bearer | 组织架构 |
| 13 | 部门详情（批量） | `/openapi/thirdparty/api/org/v1/organizations/list` | POST | Bearer | 组织架构 |
| 14 | 获取部门字段设置 | `/openapi/thirdparty/api/org/v1/metadata` | GET | Bearer | 组织架构 |
| 15 | 获取公司职位清单V2 | `/openapi/thirdparty/api/org/v1/organizations/positions/search` | POST | Bearer | 组织架构 |
| 16 | 部门下的职位清单 | `/openapi/thirdparty/api/org/v1/organizations/positions` | GET+Query | Bearer | 组织架构 |
| 17 | 根据职位id获取职位清单 | `/openapi/thirdparty/api/org/v1/organizations/positionIds` | POST | Bearer | 组织架构 |
| 18 | 根据职位id获取适用职级 | `/openapi/thirdparty/api/org/v1/organizations/positions/getPositionGrade` | POST | Bearer | 组织架构 |
| 19 | 获取职务清单 | `/openapi/thirdparty/api/org/v1/organizations/jobTitles` | GET | Bearer | 组织架构 |
| 20 | 获取职级清单 | `/openapi/thirdparty/api/org/v1/organizations/positiongrades` | GET | Bearer | 组织架构 |
| 21 | 获取体系表清单 | `/openapi/thirdparty/api/org/v1/gradesystem/all` | GET | Bearer | 组织架构 |
| 22 | 获取法律实体清单v3 | `/openapi/thirdparty/api/org/v1/corporations/search` | POST | Bearer | 组织架构 |
| 23 | 获取工作地点清单 | `/openapi/thirdparty/api/org/v1/companysites` | GET | Bearer | 组织架构 |
| 24 | 获取职务分类清单 | `/openapi/thirdparty/api/org/v1/jobCategory/get` | GET | Bearer | 组织架构 |
| 25 | 查询多维组织 | `/openapi/thirdparty/api/dimensionOrganization/v1/dimension/organizations` | GET+Query | Bearer | 组织架构 |
| 26 | 获取删除员工清单 | `/openapi/thirdparty/api/staff/v1/staffs/deleted` | GET+Query | Bearer | 员工管理 |
| 27 | 获取复职员工ID清单 | `/openapi/thirdparty/api/staff/v1/staffs/reinstatedAdditions` | GET+Query | Bearer | 员工管理 |
| 28 | 批量获取员工企业内任职记录 | `/openapi/thirdparty/api/staff/v1/employmentRecords` | POST | Bearer | 员工管理 |
| 29 | 获取教育经历清单 | `/openapi/thirdparty/api/staff/v1/educations` | POST | Bearer | 员工管理 |
| 30 | 获取工作经历清单 | `/openapi/thirdparty/api/staff/v1/experiences` | POST | Bearer | 员工管理 |
| 31 | 获取员工证照清单 | `/openapi/thirdparty/api/staff/v1/certificates` | POST | Bearer | 员工管理 |
| 32 | 获取员工银行卡V2 | `/openapi/thirdparty/api/staff/v2/staffBank/list` | GET+Query | Bearer | 员工管理 |
| 33 | 获取员工直属上级清单 | `/openapi/thirdparty/api/staff/v1/superiors` | POST | Bearer | 员工管理 |
| 34 | 获取员工兼任清单 | `/openapi/thirdparty/api/staff/v1/assignedPositions` | POST | Bearer | 员工管理 |
| 35 | 获取员工当前合同 | `/openapi/thirdparty/api/staff/v1/contracts/current` | GET+Query | Bearer | 员工管理 |
| 36 | 获取自定义信息子集记录 | `/openapi/thirdparty/api/staff/v1/subset` | POST+Query | Bearer | 员工管理 |
| 37 | 获取子集元数据定义 | `/openapi/thirdparty/api/staff/v1/subset/metadata` | GET | Bearer | 员工管理 |
| 38 | 获取入职字段定义 | `/openapi/thirdparty/api/staff/v1/entrys/fields` | GET | Bearer | 员工管理 |
| 39 | 获取待入职清单 | `/openapi/thirdparty/api/staff/v1/entryForms` | GET+Query | Bearer | 员工管理 |
| 40 | 获取待入职详情 | `/openapi/thirdparty/api/staff/v1/entryForms/{entryFormId}/entryInfo` | GET+Path | Bearer | 员工管理 |
| 41 | 根据转正日期获取转正清单 | `/openapi/thirdparty/api/staff/v1/positiveForms/positive` | GET+Query | Bearer | 员工管理 |
| 42 | 获取当日已转正员工清单 | `/openapi/thirdparty/api/staff/v2/positiveForms/positive` | GET+Query | Bearer | 员工管理 |
| 43 | 根据员工id获取待转正清单 | `/openapi/thirdparty/api/staff/v1/positiveForms/findStaffPendingForms` | POST | Bearer | 员工管理 |
| 44 | 获取待调动清单 | `/openapi/thirdparty/api/staff/v1/transferForms` | GET+Query | Bearer | 员工管理 |
| 45 | 获取时间段内调动清单 | `/openapi/thirdparty/api/staff/v1/transferWithinInterval` | GET+Query | Bearer | 员工管理 |
| 46 | 获取调动设置 | `/openapi/thirdparty/api/staff/v1/transferForms/setting` | GET | Bearer | 员工管理 |
| 47 | 获取待离职清单 | `/openapi/thirdparty/api/staff/v1/quitForms` | GET+Query | Bearer | 员工管理 |
| 48 | 获取当日离职清单 | `/openapi/thirdparty/api/staff/v1/quitForms/quited` | GET+Query | Bearer | 员工管理 |
| 49 | 获取时间段内离职清单 | `/openapi/thirdparty/api/staff/v1/quitForms/quitedWithinInterval` | GET+Query | Bearer | 员工管理 |
| 50 | 获取绩效档案清单 | `/openapi/thirdparty/api/staff/v1/staff/performances/staffIds` | POST | Bearer | 员工管理 |
| 51 | 获取家庭成员清单 | `/openapi/thirdparty/api/staff/v1/familyMembers` | POST | Bearer | 员工管理 |
| 52 | 查询员工任职记录（多维组织） | `/openapi/thirdparty/api/staffDimensionOrg/v1/staffDimensionOrg/{staffId}` | GET+Path+Query | Bearer | 员工管理 |
| 53 | 考勤数据导入 | `/openapi/thirdparty/api/payroll/v1/importAttendance` | POST | Bearer | 薪资管理 |
| 54 | 查询考勤数据导入动态表头 | `/openapi/thirdparty/api/payroll/v1/attendance/queryAttendanceFlexPart` | GET | Bearer | 薪资管理 |
| 55 | 获取公司薪资方案列表 | `/openapi/thirdparty/api/payroll/v1/queryAllSalaryPlan` | GET | Bearer | 薪资管理 |
| 56 | 获取公司薪资方案详情 | `/openapi/thirdparty/api/payroll/v1/queryAttendanceSalaryPlan` | POST | Bearer | 薪资管理 |
| 57 | 基于方案ID查询核算中薪资核算表V2 | `/openapi/thirdparty/api/payroll/v2/salaryTask/querySalary` | POST | Bearer | 薪资管理 |
| 58 | 薪资核算根据审批id查询审批文件地址 | `/openapi/thirdparty/api/payroll/v1/salaryTask/getFileUrlByProcessId` | GET | Bearer | 薪资管理 |
| 59 | 员工薪资调整 | `/openapi/thirdparty/api/payroll/v1/staffSalaryProfiles/{staffId}` | **PUT** | Bearer | 薪资管理 |
| 60 | 获取员工薪资档案调整记录 | `/openapi/thirdparty/api/payroll/v1/salaryProfile/queryWithStaffId` | POST | Bearer | 薪资管理 |
| 61 | 获取员工个税扣缴义务人集合 | `/openapi/thirdparty/api/payroll/v1/salaryStaffCorporation/query` | GET | Bearer | 薪资管理 |
| 62 | 获取员工所有个税扣缴义务人和薪资档案 | `/openapi/thirdparty/api/payroll/v1/salaryProfile/queryAllWithStaffId` | POST | Bearer | 薪资管理 |
| 63 | 薪资结果定义列表 | `/openapi/thirdparty/api/payroll/v1/queryReportDefinedList` | GET | Bearer | 薪资管理 |
| 64 | 薪资结果定义详情 | `/openapi/thirdparty/api/payroll/v1/queryReportDefinedDetail` | GET | Bearer | 薪资管理 |
| 65 | 基于部门ID和月份查询薪资结果表 | `/openapi/thirdparty/api/payroll/v1/querySalaryWithDepartment` | POST | Bearer | 薪资管理 |
| 66 | 基于人员MobileNo列表和月份查询薪资结果表 | `/openapi/thirdparty/api/payroll/v1/querySalaryWithMobileNo` | POST | Bearer | 薪资管理 |
| 67 | 基于人员ID列表和月份查询薪资结果表 | `/openapi/thirdparty/api/payroll/v1/querySalaryWithStaffId` | POST | Bearer | 薪资管理 |
| 68 | 系统薪资项目列表 | `/openapi/thirdparty/api/payroll/v1/sysSalaryField/list` | GET | Bearer | 薪资管理 |
| 69 | 根据薪资项目获取员工月度薪资结果 | `/openapi/thirdparty/api/payroll/v1/salaryResult/list` | POST | Bearer | 薪资管理 |
| 70 | 获取应用于员工薪资档案的薪资项目 | `/openapi/thirdparty/api/payroll/v1/companySalaryField/getUseInStaffSalaryFieldList` | GET | Bearer | 薪资管理 |
| 71 | 获取公司所有薪资项目 | `/openapi/thirdparty/api/payroll/v1/getAllSalaryProfile` | GET | Bearer | 薪资管理 |
| 72 | 社保缴纳方案集合 | `/openapi/thirdparty/api/insurance/v1/CompanyBenefits` | GET | Bearer | 福利管理 |
| 73 | 社保方案获取基数字段 | `/openapi/thirdparty/api/insurance/v1/payBaseHeaders/query/SI` | GET | Bearer | 福利管理 |
| 74 | 公积金方案获取基数字段 | `/openapi/thirdparty/api/insurance/v1/payBaseHeaders/query/HF` | GET | Bearer | 福利管理 |
| 75 | 其他福利方案获取基数字段 | `/openapi/thirdparty/api/insurance/v1/payBaseHeaders/query/OTHER` | GET | Bearer | 福利管理 |
| 76 | 基于年月和人员ID查询已存档社保台账结果 | `/openapi/thirdparty/api/insurance/v1/staffMonthlyLedger/list` | POST | Bearer | 福利管理 |
| 77 | 获取休假单 | `/openapi/thirdparty/api/tm/v1/vacations/orders/search` | POST | Bearer | 考勤管理 |
| 78 | 获取假期类型 (v1) | `/openapi/thirdparty/api/tm/v1/vacations/types/search` | GET | Bearer | 考勤管理 |
| 79 | 获取所有假期类型 (v2) | `/openapi/thirdparty/api/tm/v2/vacations/types/search` | GET | Bearer | 考勤管理 |
| 80 | 获取一个员工假期余额 | `/openapi/thirdparty/api/tm/v1/vacations/limit/search` | POST | Bearer | 考勤管理 |
| 81 | 获取法定节假日 | `/openapi/thirdparty/api/tm/v1/vacations/statutoryHoliday/search` | GET | Bearer | 考勤管理 |
| 82 | 获取年假余额 | `/openapi/thirdparty/api/tm/v1/vacations/annual/limit/search` | POST | Bearer | 考勤管理 |
| 83 | 获取加班单v2 | `/openapi/thirdparty/api/tm/v2/overtimes/orders/search` | POST | Bearer | 考勤管理 |
| 84 | 获取加班设置 | `/openapi/thirdparty/api/tm/v1/overtimes/setting/query` | GET | Bearer | 考勤管理 |
| 85 | 加班单时长试算 | `/openapi/thirdparty/api/tm/v1/overtimes/duration/calc` | POST | Bearer | 考勤管理 |
| 86 | 根据结转日期获取加班单 | `/openapi/thirdparty/api/tm/v1/overtimes/orders/carryTime/search` | POST | Bearer | 考勤管理 |
| 87 | 获取外出单V2 | `/openapi/thirdparty/api/tm/v2/fieldwork/orders/search` | POST | Bearer | 考勤管理 |
| 88 | 获取出差单V2 | `/openapi/thirdparty/api/tm/v2/businesstravel/orders/search` | POST | Bearer | 考勤管理 |
| 89 | 获取补卡原因 | `/openapi/thirdparty/api/tm/v1/appealReason/getReasonByStaffId` | GET | Bearer | 考勤管理 |
| 90 | 获取补卡单 | `/openapi/thirdparty/api/tm/v1/reissues/orders/search` | POST | Bearer | 考勤管理 |
| 91 | 获取打卡记录 | `/openapi/thirdparty/api/tm/v2/signs/records/search` | POST | Bearer | 考勤管理 |
| 92 | 获取排班V4 | `/openapi/thirdparty/api/tm/v4/schedules/actual/all` | POST | Bearer | 考勤管理 |
| 93 | 获取日报V4 | `/openapi/thirdparty/api/tm/v4/daily/reports/search` | POST | Bearer | 考勤管理 |
| 94 | 获取月报V3 | `/openapi/thirdparty/api/tm/v3/period/monthly/reports/search` | POST | Bearer | 考勤管理 |
| 95 | 获取考勤周期设置 | `/openapi/thirdparty/api/tm/v1/attendance/periods/settings/search` | GET | Bearer | 考勤管理 |
| 96 | 获取日报自定义字段 | `/openapi/thirdparty/api/tm/v1/custom/field/list/search` | GET | Bearer | 考勤管理 |
| 97 | 获取员工考勤周期设置 | `/openapi/thirdparty/api/tm/v1/attendance/periods/staff/settings/search` | POST | Bearer | 考勤管理 |
| 98 | 根据考勤周期id获取员工设置 | `/openapi/thirdparty/api/tm/v1/attendance/periods/staff/settings/searchBySettingId` | POST | Bearer | 考勤管理 |
| 99 | 获取考勤周期实例 | `/openapi/thirdparty/api/tm/v1/attendance/periods/instance/search` | POST | Bearer | 考勤管理 |
| 100 | 获取日报自定义字段值 | `/openapi/thirdparty/api/tm/v1/daily/custom/search` | POST | Bearer | 考勤管理 |
| 101 | 获取月报出勤班次 | `/openapi/thirdparty/api/tm/v1/period/monthly/shift/search` | POST | Bearer | 考勤管理 |
| 102 | 获取月报自定义字段 | `/openapi/thirdparty/api/tm/v1/period/custom/field/list/search` | GET | Bearer | 考勤管理 |
| 103 | 获取第三方选项数据GET | OA回调（URL由第三方提供） | GET | 自定义 | 审批相关 |
| 104 | 获取第三方选项数据POST | OA回调（URL由第三方提供） | POST | 自定义 | 审批相关 |
| 105 | 通过流程id和审批人id获取待办 | `/openapi/thirdparty/api/workflow/v1/process/task/backLog/{processId}` | POST | Bearer | 审批相关 |
| 106 | 根据参数查询审批已完成的单据数据 | `/openapi/thirdparty/api/workflow/v1/process/finish/page` | POST | Bearer | 审批相关 |
| 107 | 根据流程id修改审批流程状态 | `/openapi/thirdparty/api/workflow/update/process/status` | POST | Bearer | 审批相关 |
| 108 | 根据标记码变更流程实例数据 | `/openapi/thirdparty/api/workflow/v1/form/update/instance` | POST | Bearer+JWT | 审批相关 |
| 109 | 通过流程ID审批节点审批人审批状态执行审批任务 | `/openapi/thirdparty/api/workflow/v1/process/complete/task` | POST | Bearer | 审批相关 |
| 110 | 查询操作日志 | `/openapi/thirdparty/api/operationlogs/v1/api/query/ids` | POST | Bearer | 操作日志 |
| 二方-111 | 假期设置-限制规则 | `/web/gateway/attendance/api/leaveSetting/...` | GET | Cookie+XSRF | Gateway |
| 二方-112 | 额度显示类型 | `/web/gateway/attendance/api/limitutil/...` | GET | Cookie+XSRF | Gateway |
| 二方-113 | 额度单假期类型 | `/web/gateway/attendance/api/limitBill/vacationType` | GET | Cookie+XSRF | Gateway |
| 二方-114 | 额度单列表 | `/web/gateway/attendance/api/limitBill/list` | GET | Cookie+XSRF | Gateway |
| 二方-115 | 使用记录列表 | `/web/gateway/attendance/api/record/list` | GET | Cookie+XSRF | Gateway |
| 二方-116 | 额度变更记录 | `/web/gateway/attendance/api/limitBill/record` | GET | Cookie+XSRF | Gateway |
| 二方-117 | 转薪资结余记录 | `/web/gateway/attendance/api/limitBill/transferSalaryRecord` | GET | Cookie+XSRF | Gateway |
| 二方-流程定义 | 审批类型设置-流程定义 | `/gateway/workflow/aggregate/approval/setting/draft/{id}` | GET | Cookie+XSRF/宿主 | 审批相关 |
| 二方-分组 | 审批类型分组列表 | `/gateway/workflow/aggregate/activiti/common/approvalGroup/selector` | GET | Cookie+XSRF/宿主 | 审批相关 |
| 二方-类型列表 | 审批设置-分组及类型列表 | `/gateway/workflow/aggregate/approval/setting/findApprovalGroupVo` | GET | Cookie+XSRF/宿主 | 审批相关 |
| 二方-解锁 | 未发布草稿查询 | `/gateway/workflow/aggregate/approval/setting/hasUnPublishDraft/{id}` | GET | Cookie+XSRF/宿主 | 审批相关 |
| OpenAPI-员工分组 | 审批流权限分组-员工部门范围 | `/openapi/thirdparty/api/auth/v1/group/approval` | POST | Bearer | 审批相关 |
| 二方-薪资公式 | 薪资项目列表含公式 | `/gateway/payroll/api/companySalaryfieldMetas/list` | POST | Cookie+XSRF/宿主 | Gateway |
| 二方-薪资解析 | 薪资项目公式解析 | `/gateway/component/manage/api/component/table/manage/enable` | GET | Cookie+XSRF/宿主 | Gateway |
