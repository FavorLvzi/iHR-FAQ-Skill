# 问题类型索引（problem-index，2026-08-12 新增）——按问题快速定位

> **用法**：用户提问后先判问题类型 → 查下表定位入口文档 → 按入口文档的方法论回答。
> 接口规格统一在 `references/api-handbook/`（按模块 01~09 组织，编号索引见 index.md）。

## 场景化入口总表

| 问题类型 | 典型问法 | 入口文件（★主入口） | 关键接口/数据 |
|---------|---------|-------------------|--------------|
| **考勤异常排查** | "为什么缺勤/实际出勤不对"、"打不了卡不在范围" | `references/attendance-field-logic.md` 三·八（双视角+打卡前后分流+鱼骨图） | #91 打卡 / #93 日报 / #110 操作日志 / #96 公式 / 排班 |
| **打卡前被拦**（功能用不了） | "无法打卡/没有打卡入口" | `references/fishbone-method.md`（鱼骨图①不在范围 ②无入口） | 规则配置咨询（无需 API） |
| **考勤字段计算** | "应出勤/实际出勤/缺勤怎么算" | `references/attendance-field-logic.md` 三·一~三·五 | #91~#102（#93/#94/#96/#100/#102）+ **#96 自定义字段定义（★日报必须关联查，见 api-reference「#93 vs #96」）** |
| **假期额度** | "假期余额怎么算/为什么少了" | `references/attendance-field-logic.md` 三·五 + `references/api-reference.md` | #79 规则 / #80 余额 / #82 年假明细 / 二方 #116 流水 |
| **加班转薪资** | "转薪资额度够不够/锁定逻辑" | `references/attendance-field-logic.md` 三·六（净锁定判定） | #83 加班单 / #86 结转 / #117 锁定账本 / #OL 日志 |
| **薪资公式解读** | "某薪资项怎么算的" | `references/formula-language.md`（70+ 函数语法） | 二方 `companySalaryfieldMetas/list`（公式）/ #67 值验证 |
| **流程走向分析** | "某审批怎么走/谁审批/走哪条分支" | `references/attendance-field-logic.md` 三·九 | `findApprovalGroupVo` → `approval/setting/{id}`（已发布） |
| **具体员工走向**（★高频） | "员工X 的补卡怎么走" | `api-handbook/08-审批相关/审批流程-可视化生成.md`「★具体员工走向分析」 | + `auth/v1/group/approval` 分组 + #2 详情 → **必配 Mermaid 图** |
| **功能操作指引** | "XX 功能怎么用" | `references/attendance-field-logic.md` + 语雀锚点（sources.md） | 文档锚点（help/faq）优先 |

## 数据获取速查（先 OAuth 授权，再按需取数）

| 数据需求 | OpenAPI（OAuth） | 二方 Gateway（宿主 token，细则见 output-rules.md） |
|---------|-----------------|--------------------------------------------------|
| 员工 | #1 ID 清单 / #2 详情 / 花名册搜索 #5 | — |
| 考勤日报/月报 | #91~#102（#93/#94/#96/#100/#102） | — |
| 假期 | #77 休假单 / #79 规则 / #80 余额 / #82 年假 | #111~#116（#116 额度变更流水★） |
| 加班 | #83 加班单 / #86 结转 | #117 锁定账本 |
| 薪资 | #63 方案 / #64 归属 / #67 月度核算 | `companySalaryfieldMetas/list`（公式）+ 公式解析 |
| 审批流程 | #105 待办 / #106 已完结单据 | `findApprovalGroupVo` / `approval/setting/{id}` / `hasUnPublishDraft`+`draft` |

> ★ #106 查空排查清单 / 流程定义必须取最新（禁历史单据拼接）见 `references/output-rules.md`。
