# API 实操陷阱速查（★2026-08-10 宿主 AI 实测沉淀）

> 宿主 AI 在 i人事内置 AI 环境中实测 OAuth 授权流程时踩到的 4 个坑。**这些不是"skill 该怎么做"的问题**——OAuth 授权流程和接口路径描述本身是准确的——而是 API 的实际运行行为与文档描述有细微差异，需要试错才能定位。本文件把这些经验沉淀下来，让后续宿主 AI 少走弯路。

## 陷阱 1：域混淆（截图提及，skill 已规避）

- **陷阱**：API 手册示例写 `api.ihr360.com`，实际端点是 `openapi.ihr360.com`，用错域名连接失败
- **正确**：所有 OpenAPI（#1~#110 等）都用 **`openapi.ihr360.com`**；二方 Gateway 接口（draft / findApprovalGroupVo 等）用 **`v5.ihr360.com`**
- **skill 内已统一**：所有接口文档 Base URL 均用 `openapi.ihr360.com`（已通过 grep 验证 0 处错误域）

## 陷阱 2：#106 传了 companyId 可能无权访问（★需注意）

- **陷阱**：传 companyId 但应用未授权该公司 → "无权访问的公司id"错误
- **正确**：OAuth 授权时绑定的应用 = 应用可访问的公司列表；customer 在 `open.ihr360.com` 创建应用时已指定公司范围
- **如果遇到**：① 检查应用管理中的"可访问公司"配置 ② 或调用应用授权接口扩展权限

## 陷阱 3：#106 statuses 数组传了多元素 → JSON 解析错误

- **陷阱**：传数组 `["PASS", "DENIED"]` → 触发 JSON 解析错误（"code=-1"），原因是接口**只接受单元素数组**
- **正确**：`statuses: ["PASS"]` 单值；若想查多种状态，**分多次调用**（按 flowPriority 处理不同问题）
- **官方文档措辞**："仅限已完结状态"——未明确"单值"，实测确认后写明
- **关联接口**：`auth/v1/group/approval` 也只接受单页（page+size），不返回多页合并

## 陷阱 4：路径版本试错（staff/v2 vs staff/v1）

- **陷阱**：猜路径 `/staff/v2/staffs/details` → 404，正确是 `/staff/v1/staffs/{id}/detail`
- **正确**：
  - 员工详情：`/openapi/thirdparty/api/staff/v1/staffs/{staffId}/detail`（V1）
  - 员工银行卡：`/openapi/thirdparty/api/staff/v2/staffBank/list`（V2，**注意：模块不同版本不同**）
  - 当日已转正员工：`/openapi/thirdparty/api/staff/v2/positiveForms/positive`（V2）
- **★规律**：同模块的接口**可能混合 v1 和 v2**——`staffs` 主体 v1，`staffBank`/`positiveForms` 子模块 v2
- **避坑**：调接口前**先查 `api-handbook/index.md`**，里面有完整接口编号 + 路径 + 版本索引

## 实战避坑 SOP

```
1. 拿到问题 → 查 api-handbook/index.md 按编号找接口
2. 确认路径（注意 v1/v2 子模块差异）
3. 确认必填参数（公司 ID / 公司编码 / 状态枚举等）
5. 第一次调用前：所有字段都用全默认值（最少必要字段），避免漏必填
6. 报错后：先看 code 与 message，对照本文档排查
   - 401 → 鉴权/token 过期
   - 403 → 跨域（openapi vs v5）/ 跨公司
   - 404 → 路径版本错
   - code=-1 + JSON 错误 → 多半是 statuses/page 等数组型参数传了多元素
```

## 沉淀来源

- 2026-08-10 宿主 AI 实测 OAuth 授权 + 员工 ID 流程的反馈（截图）
- 截图结论："这些都不是 skill 该怎么做的问题，而是 API 的实际运行行为与文档描述有细微差异，需要试错才能定位。skill 的 OAuth 授权流程和接口路径描述本身是准确的。"
- 后续每发现一个坑补充进本文档