# 内容锚点来源规则

本技能所有口径事实都必须能追溯到以下权威来源。

## 唯一认可的锚点地址

| 地址 | 用途 | 特点 |
|------|------|------|
| `https://ihr360.yuque.com/ihr/help/` | 用户手册（配置逻辑 / 计算口径） | 结构清晰、slug 稳定、内容详尽，**主源** |
| `https://ihr360.yuque.com/ihr/faq` | 常见问题（操作 / 报错视角） | 抓取不稳定，作辅助补充 |

## 硬性约束

1. 口径事实**只引用** `https://ihr360.yuque.com/ihr/help/` 与 `https://ihr360.yuque.com/ihr/faq` 及其子页。
2. 引用时**锚定到用户指定的两个语雀地址**：`help/`、`faq` 及子页（`ihr360.com/hrnews`、`blog.ihr360.com` 等官网博客非指定锚点，口径可能不一致，不采用）。
3. 子页 URL **多为随机串**（`help/kq`、`help/hwgsov`），引用前**先抓取页面定位到准确 URL** 再转述原文，不依据页面名猜测内容。
4. 若某字段逻辑在 help/faq 中找不到锚点，**明确告知用户**"未在指定源中找到"，并给出去哪查（如对应系统的"考勤设置 → 复杂场景规则设置"界面）。

## 已验证可读取的子页 slug

- `help/kq` —— 考勤管理（含班次、考勤设置、复杂场景规则设置等子页入口）
- `help/hwgsov` —— 日报字段处理（自定义字段引擎约束：循环引用、字段启用后不可逆、91 天限制、重算边界）
- `help/fnuk248zb9trbgh3` —— **APP/三方应用设置**（账号→基础设置→APP和ESS设置；应用管理=配置 APP 展示的应用/排序，首页设置=默认插件，工作台设置；**打卡应用是否在员工 app 可见=打卡入口缺失的排查锚点**，2026-08-07 验证）

> 注：`help/faq` 根页抓取有时不稳定，可多次尝试或让用户直接提供具体子页 URL。

## ★ 根页导航能力边界（2026-08-13 实测）

| 页面 | 能否抓取 | 能拿到什么 |
|------|---------|-----------|
| 根页 `ihr360.yuque.com/ihr/help` | ✅ 可抓 | **完整一级目录**（全部模块链接：考勤=`kq`、薪资=`xz`、审批=`sp`、组织人事=`kgk8gd`、社保=`fl`、招聘=`gdviog`、绩效=`jx`、工作台=`pfih8g`、账号=`on9nxn` 等） |
| 分类页（如 `help/kq`） | ✅ 可抓 | 功能地图/特色功能/系统介绍正文；**但侧边栏深层目录（具体文档链接）抓不到**——语雀目录是 JS 渲染，WebFetch 提取不到 |
| 具体文档页（如 `help/hwgsov`） | ✅ 可抓 | 完整正文（前提：知道子页 slug） |

**结论**：只知道根网址时——
1. ✅ 能拿到**一级目录**（知道有哪些模块 + 各模块 slug）
2. ⚠️ **无法自动导航到具体文档**（深层链接 JS 渲染抓不到）
3. 因此定位具体文档正文的方式：**① 用上表/本文件已验证 slug 直接抓；② 让用户提供子页 URL；③ 用系统内的"帮助"搜索入口由用户代查**

## ★★ 官方 FAQ 搜索接口（query_faq，★首选，2026-08-13 整合）

**按问题搜索官方知识库，直接返回语雀文档链接 + 内容**——无需知道 slug、无需导航目录，比 WebFetch 更可靠。**口径类问题优先用此接口**，WebFetch 作补充（抓已验证 slug 的深度正文）。

### 接口规格

| 项目 | 值 |
|------|-----|
| 端点 | `POST https://v5.ihr360.com/gateway/digital/robot/digital/intelligent/assistant/v2/query_faq` |
| 鉴权 | `Authorization: Bearer {IHR360_API_TOKEN}`（宿主环境变量自动注入，**勿硬编码 token**） |
| Content-Type | `application/json; charset=utf-8` |

| 参数 | 类型 | 说明 |
|------|------|------|
| `content` | string | 用户问题（必填） |
| `answerCount` | int | 返回条数，默认 3 |
| `inside` | bool | 默认 false |
| `onlyStaff` | bool | 默认 false |
| `recognizeIntention` | bool | 默认 true |

### 返回结构（data[]）

| 字段 | 说明 |
|------|------|
| `name` | 文档标题 |
| `linkUrl` | 语雀文档链接（`https://www.yuque.com/ihr/help/xxx`） |
| `_cleanContent` | 清理后的纯文本正文 |
| `_cleanSearchContent` | 匹配片段 |
| `_images` | 图片 URL 列表（语雀 CDN，**非空时必须内联 Markdown 展示**） |
| `normalizedScore` | 匹配度分数 |

### 调用示例（宿主 AI）

```bash
curl -s -X POST "https://v5.ihr360.com/gateway/digital/robot/digital/intelligent/assistant/v2/query_faq" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Authorization: Bearer ${IHR360_API_TOKEN}" \
  -d '{"content":"日报自定义字段怎么处理？","answerCount":3,"inside":false,"onlyStaff":false,"recognizeIntention":true}'
```

### 输出呈现规范（4 步）

1. **概览表格**：序号 | 标题 | 匹配度 | 来源链接（[查看](linkUrl)）
2. **关键图片**：`_images` 非空 → 内联 Markdown 展示操作截图
3. **内容摘要**：从 `_cleanContent` 提取与问题最相关的操作步骤/规则（≤300 字）
4. **来源链接**：末尾附语雀文档链接

### 使用流程

1. 口径/规则类问题 → **先调 query_faq**（content=用户问题）
2. 结果含 linkUrl + 内容 → 按 4 步呈现；linkUrl 即锚点来源（与"唯一认可锚点"一致）
3. 搜不到/需深度正文 → **降级 WebFetch** 抓已验证 slug（`help/kq`、`help/hwgsov` 等）或请用户提供子页 URL
