# authority 权限域 · 用户数据授权（dataAuth，★数据范围 完整版）

> **2026-08-18 用户抓包全量**——数据授权 = **基础数据范围**（basic）+ **功能级数据范围**（authorityTree 每节点可配）。读：`webUser/dataAuth`，写：`row_data/userDataAuth`。

## ① 读：查询数据授权（GET）

`GET /gateway/authority/aggregate/webUser/dataAuth?webUserId={userId}`

## ② 写：保存数据授权（POST）

`POST /gateway/authority/api/auth/row_data/userDataAuth?webUserId={userId}`（body = 下方结构，待 Payload 确认）

## 响应完整结构（两层模型）

### 第一层：基础数据授权（basic，用户级全局）

| 字段 | 说明 |
|------|------|
| `departCheckAll` / `departIds[]` | 部门范围：全部 / 指定部门（**值为部门 dwAgentKey**，如 [96,113,66,98,84,101,117,87,104,110]） |
| `contractCorpCheckAll` / `contractCorpIds[]` | 合同公司范围：全部 / 指定公司（**值为公司 UUID**） |
| `siteCheckAll` / `siteIds[]` | 工作地点范围 |
| `includeGroups[]` / `excludeGroups[]` | 包含/排除**员工分组**：`{groupId, groupName, check}` |
| `basicDataAuthNodeTypes[]` | 可用节点类型枚举 |
| `bindingInnerRoleCodes[]` | 绑定的内置角色编码 |
| `hasSeniorSetting` | 是否有高级设置 |

### 第二层：功能级数据授权（authorityTree，每功能节点可配）

```
authorityTree[] = 功能权限树（与菜单 authorityId 对应）★树始终存在（全功能树）
├── id: "112" / authorityName: "员工" / functionCode: "staffManage" / functionLayer: 1
│   └── children: id: "11202" "异动管理" / functionCode: "staffManage.relationship"
│       └── children: id: "1120201" "入职" / functionCode: "staffManage.entryManage"
│           ├── authorityDataRow: {departIds, contractCorpIds, siteIds, includeGroups, excludeGroups}
│           │     ★为空数组=该功能节点未单独配置（跟随基础层）
│           ├── dataAuthNodeTypes: [TREE, CONTRACT_CORP, SITE, INCLUDE, EXCLUDE]（部分节点含 DATA_TYPE）
│           ├── nodeType2Name: {TREE: "选择部门", CONTRACT_CORP: "选择合同公司", SITE: "选择工作地点", INCLUDE: "选择员工分组", EXCLUDE: "排除员工分组"}
│           ├── followBasic: true          ← 跟随基础设置（true=用基础层范围）
│           ├── hasNoAuthorityData: true   ← 无数据授权（★判断"是否配置"用此标记）
│           └── dataAuthLimitByBasic: false  ← 是否受基础限制
```

**★ 正确解读（2026-08-18 实测修正）**：
- `authorityTree` **始终是完整功能树**（不能以"节点数=0"判断——那是"有数据配置的节点数"）
- 判断用户数据权限是否配置：
  1. **基础层** departIds/contractCorpIds/siteIds 是否有值（null/空 = 未配置）
  2. **功能级** authorityDataRow.departIds 是否非空（空数组 = 未单独配置，followBasic=true 跟随基础层）
  3. `hasNoAuthorityData: true` = 该节点无数据授权
- **实测（员工D/员工X）**：基础层全部空 + 功能级全部 followBasic=true 且 authorityDataRow 空 → **实际无数据授权配置**（`dataAuths.authorized=true` 来自角色级，非用户级）

**节点类型枚举**：`DATA_AUTH_ID_TYPE_TREE`（选择部门）/ `CONTRACT_CORP`（选择合同公司）/ `SITE`（选择工作地点）/ `INCLUDE`（选择员工分组）/ `EXCLUDE`（排除员工分组）/ `ABSTRACT_DEPT`（抽象部门）/ **`DATA_TYPE`（数据类型，考勤/薪资等节点专属，2026-08-18 实测补）**

## 数据源接口（选择器，★授权时的选项来源）

| 接口 | 用途 | 返回关键字段 |
|------|------|------------|
| `POST /gateway/component/api/organization/v1/treeSelector` | **组织树选择器**（选部门） | 公司/部门树：`name / departmentId / dwAgentKey（=departmentId，数据授权传参值）/ departmentCode / departmentStatus / children / principalName` |
| `POST /gateway/component/api/organization/v1/Corporation/getALL` | **合同公司列表**（选合同主体） | `id（UUID）/ companyName` |

## ★★ 用户数据授权保存链路（数据来源 → 结构 → 保存）

```
① treeSelector 拉组织树（dwAgentKey=部门ID）
② Corporation/getALL 拉合同公司（id=UUID）
③ 组装：基础层（departIds/contractCorpIds/siteIds/includeGroups/excludeGroups）
        + 功能层（authorityTree[] 每节点 authorityDataRow）
④ row_data/userDataAuth(webUserId, 上述结构) → data:true 保存
```

## ★★★ 人员数据授权闭环（2026-08-18 确认，查 2 + 存 1）

| 接口 | 角色 | 实例验证 |
|------|------|---------|
| `component/api/organization/v1/treeSelector` | **部门树选项**（dwAgentKey=departIds 值） | 全公司树，dwAgentKey=departmentId |
| `aggregate/webUser/dataAuth?webUserId=` | **已授权范围（读现状）——⚠️仅浏览器会话可用，HTTP 直调返回空模板（见下「★★★ 读取限制」）** | 浏览器抓包示例：员工D departIds=[16,19,4,10,13]（产品部/人力资源部/四川淘金/研发部/运维部，与 treeSelector 交叉验证） |
| `api/auth/row_data/userDataAuth` | **保存（写）** | data:true |

> **交叉验证**：dataAuth 的 departIds 值 = treeSelector 树的 dwAgentKey（departmentId）——同一个部门 ID 体系。
> **授权范围四维度**：部门（departIds）+ 合同公司（contractCorpIds）+ 工作地点（siteIds）+ 员工分组（includeGroups/excludeGroups）——全部在 dataAuth 读响应中体现。

## ★★★ 读取限制（2026-08-18 三组实验定案，★重要）

**`aggregate/webUser/dataAuth` 读接口：完整数据仅真实浏览器会话可用；HTTP 直调（Python 复制 Cookie / 宿主 API 代理）一律返回降级空模板。**

三组实验证据（员工D，浏览器已配置 departIds=[16,19,4,10,13]）：
1. **路径前缀对比**：`/gateway/` 与 `/web/gateway/` 均返回空（departIds=None）
2. **完整 Cookie + 完整 header**（含 TV_CM/IHR_TOKEN_V1/浏览器 UA/Referer）→ 仍空
3. **模拟浏览器页面初始化序列**（user/me→菜单树→subHr/list→listByIds 预热）→ 仍空；且**降级响应连 authorityTree 都是 []**（浏览器返回完整功能树）——**证明不是"配置查不到"，而是接口对非浏览器调用返回空模板**

**结论**：
- 浏览器打开用户管理页面 → dataAuth 返回**完整结构**（authorityTree 功能树 + departIds + hasSeniorSetting=true）
- Python / 宿主 HTTP 直调 → 返回**空模板**（authorityTree=[] + departIds=None）——**不可用于判断数据权限是否配置**
- 宿主 AI 若直接 HTTP 调用同样降级（实测宿主也查不到）——**除非宿主用浏览器自动化（agent-browser）打开页面抓取**

**数据权限现状查询的可用替代（★API 可查）**：
1. **概览状态**：`simpleUser/subHr/list` 响应里的 `dataAuths`（`authorized / hsaAll / admin / it`）——判断"是否已授权数据权限"（实测员工X/员工D均返回 authorized=true,hsaAll=false）✅
2. **精确范围**（departIds/contractCorpIds/siteIds/分组）：仅浏览器页面可看；宿主可引导客户在页面确认，或用浏览器自动化抓取
3. **写操作** `api/auth/row_data/userDataAuth`：可用（实测 data:true），但**写前必须确认现状**（读不到现状时盲写可能覆盖已有配置）——建议先 subHr/list 看 dataAuths 概览再决定
