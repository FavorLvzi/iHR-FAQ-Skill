# 数据获取与接口调用规则（output-rules，2026-08-12 从 SKILL.md 拆出）

> 本文件收纳"数据获取/接口调用"的细则（从 SKILL.md「输出呈现规范」拆出，SKILL.md 只留入口指针）。
> **输出形态/跨月汇总/Mermaid/测试隔离/可读性转换等"输出硬规则"仍留在 SKILL.md**（随 prompt 加载）。

## 一、二方接口调用方式

宿主环境自动注入 `IHR360_API_TOKEN` 环境变量（格式 `sk-irs-*`），用于调用 Gateway 接口：

- Base URL: `https://v5.ihr360.com`
- Header: `Authorization: Bearer {IHR360_API_TOKEN}`
- 与 OpenAPI 的 OAuth Bearer 不互通，不可混用（混调必 403）
- 进入会话时先检查 `IHR360_API_TOKEN` 是否存在，存在则二方接口可用

## 二、二方接口可用性速查（实测，2026-08-12 更新）——调用前先查此表

| 状态 | 接口 | 说明 |
|------|------|------|
| ✅ 实测可用 | #116 额度变更记录 / #117 转薪资结余 / #113 额度单假期类型 / 花名册搜索 #5 / 薪资公式 `companySalaryfieldMetas/list` + `component/table/manage/enable` / 审批 `findApprovalGroupVo`（★定位审批类型首选，2026-08-12 事假文档实测）/ `approval/setting/{id}`（★需正确 approvalSettingId）/ `hasUnPublishDraft` + `draft/{id}` | 宿主 Bearer 直接可调 |
| ⚠️ 仅前端 Cookie（宿主 Bearer 404） | `approvalGroup/selector`（分组列表简化版，可用 findApprovalGroupVo 替代） | 不必用，findApprovalGroupVo 已覆盖 |
| ⚠️ 宿主环境 404/500（待确认路径） | #111 限制规则 / #112 额度显示类型 / #114 额度单列表 / #115 使用记录列表 | 可能需 Cookie+XSRF 或路径变体；**取不到时用 #116 或 #80/#82 降级** |
| ⚠️ 需 Cookie+XSRF（非 Bearer） | 部分表格类接口（带 `?r_id=` 的） | 宿主 Bearer 失败时，尝试前端插件 Cookie+XSRF（宿主会话自动携带） |

**调用失败时的降级链**：先查表 → Bearer 调用 → 失败换 Cookie+XSRF → 仍失败用 OpenAPI 等价接口（#80/#82/#106）→ 仍失败给用户明确说明"该接口需抓包确认路径"。

## 三、★ 流程定义必须取最新（2026-08-12 定案，勿用历史单据拼接）

正常路径：`findApprovalGroupVo`（✅ 宿主可用）→ 按 name 匹配类型 → 拿 approvalSettingId → `approval/setting/{id}` 拿**当前生效**的流程定义（节点/条件/审批人）。

- **❌ 禁止用历史审批单（#106 taskInfoList）拼接"流程走向"**——历史单据走的是**当时的旧流程**，流程定义更新后拼接结果 ≠ 当前生效流程，会误导客户（2026-08-12 定案删除该方案）
- **拿不到流程定义时**：明确告知"当前环境无法获取该审批类型的流程定义"，请用户提供 `approvalSettingId`（系统后台"审批设置→该类型"URL 里的 id）或由管理员后台查看流程配置，再调 `approval/setting/{id}`
- **#106 单据仍可用于**：确认某张单据是否审批通过、查审批意见等**单点事实**——但不能用来还原"流程怎么走"

## 四、★ #106 查空排查清单（totalElements=0 时按序排查）

> 勿轻易下"无权限/无数据"结论；#90 能查到 processId 就说明单据权限正常，此时 90% 是参数问题。

1. **page 必须从 1 开始**（page=0 返回空！此接口与多数接口分页从 0 开始不同）
2. **去掉 processUniqueId / processUniqueIds**——文档示例/skill 记录的是某环境编码（如 992311786902663168），**因环境而异**（同 approvalSettingId 同理），硬编码必然查空
3. **statuses 只传单值** `["PASS"]`（传多值报 code=-1，不是空）
4. **modelShowTypeList 枚举正确**：补卡=`APPEAL` / 出差=`EVECTION` / 假期=`VACATION`（大类，勿用具体假期名）
5. **source 显式传查询年份**（如 `"2026"`）——跨年必传，不传默认当年但有 reset 语义坑
6. **时间范围放宽**：startTimeStart 给当月月初、startTimeEnd 给月末（勿精确到日）
7. **最小查询验证接口通不通**：只传 `modelShowTypeList` + `statuses` + `page:1` + `rows:10`（不加员工/时间过滤）→ totalElements>0 说明接口通、是过滤条件问题；=0 再**去掉 modelShowTypeList 全类型试**；仍 0 才考虑该环境确实无审批单据/权限

## 五、数据获取优先级

- **首选**：按 oauth-guide.md 拿到 OAuth Bearer 后走 OpenAPI（#1~#110）
- **二方 Gateway 接口（#116/#117 等额度账本类）**：用于拿 OpenAPI 不开放的额度流水/锁定账本
- 详情见 SKILL.md「宿主登录态」与 oauth-guide.md
