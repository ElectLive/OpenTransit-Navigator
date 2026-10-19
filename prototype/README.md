# OpenTransit-Navigator 首期 UI 原型图（Phase 1 Wireframes）

> ⚠️ **AI 生成内容声明 (AI-Generated Notice)**
> 本目录下的 UI 原型图（包括视觉稿、HTML/CSS 源码）及本说明文档，均由 AI 辅助生成（AI-generated）。此设计仅作为 Phase 1 阶段的产品构想、功能推演与字段映射参考，不构成最终的技术实现标准。最终接口交互与数据结构请以实际联调的 OTP 服务为准。

本目录包含项目首期技术验证阶段的低保真 UI 线框图。设计核心目标为**信息密度优先**与**参数透明化**，摒弃商业地图大圆角、大留白的视觉风格，专为交通极客与需要深度自定义路线的用户设计。

## 🔗 设计源文件
- **[Figma 在线预览链接](https://www.figma.com/design/awdQ2NUjDFNF3jj6nolsbH/Untitled?node-id=0-1&t=0qp0yCrp10e0UTkg-1)** *(权限：Anyone with the link can view)*

## 📁 页面文件说明(注：以下字段映射为前期产品构想，具体实现以 OTP 联调结果为准)

### 1. `01-search-params.png` (搜索与参数配置页)
- **核心功能**：起终点输入、时间选择器、高级路由参数折叠面板（默认展开）。
- **UI 到数据的映射**：
  - `最大方案数量` -> 对应 OTP 参数 `numTripPatterns` (范围 10-100，默认 50)。
  - `相对额外换乘数` -> 对应 OTP 参数 `maxAdditionalTransfers` (下拉单选 5/10/15)。
  - `绕路容忍度` -> 对应 OTP 参数 `transitGeneralizedCostLimit` 及其底层的 `costLimitFunction`。
  - `显示所有被过滤方案` -> 对应 Debug 模式参数 `LIST_ALL`。

### 2. `02-list-normal.png` (方案列表页 - 普通风格)
- **核心功能**：展示海量出行方案，支持无限滚动（触底加载）。
- **UI 到数据的映射**：
  - **顶部筛选/排序**：3个排序选项（最快、最少换乘、最少步行）+ 2个筛选开关（只看直达、只要地铁）。
  - **卡片核心指标**：总耗时 `duration`、出发/到达时间、总换乘次数 `transfers`、步行总距离 `walkDistance`。
  - **路线摘要条**：对应 `legs` 数组的 `mode` 属性（步行/公交/地铁/公共自行车）。
  - **接驳位**：若包含公共自行车，卡片右下角单独高亮展示骑行时间。
  - **状态标签**：若方案被过滤（`deleted=true`），卡片右上角显示“系统已过滤”，供调试参考。

### 3. `02-list-geek.png` (方案列表页 - 极客风格)
- **核心功能**：高密度数据展示，供开发者快速核对 OTP 接口返回的原始数据。
- **UI 到数据的映射**：直接将 OTP 的 GraphQL 字段名（`duration`, `transfers`, `walk`, `bike`, `deleted`, `systemNotices`）作为表头和外露标签展示。

### 4. `03-detail-timeline.png` (路线详情与时间轴)
- **核心功能**：纵向时间轴展示完整行程，重点体现多次换乘和系统通知。
- **UI 到数据的映射**：
  - **时间轴节点**：精确映射 `legs` 数组（每一段行程的起终点、车牌/线路号 `route`、耗时）。
  - **换乘高亮（>=3次）**：换乘节点使用实心黑框警告，并展示步行与等待时间 `transferSlack`。
  - **System Notices 区**：底部独立区块，展示 JSON 返回的过滤原因（对应 `systemNotices` 字段）。例如：因比最短时间多出 17 分钟被商业算法隐藏，或广义成本超出上限。

## 🛠️ 开发组件拆分建议
基于以上设计，前端建议拆分为以下核心组件：
- `<ParamSlider />` / `<ParamSelect />` / `<DebugToggle />` (搜索页参数组件)
- `<ItineraryCard />` (列表页卡片组件)
- `<LegSummaryBar />` (列表页路线摘要条)
- `<LegTimeline />` (详情页纵向时间轴)
- `<SystemNoticeBlock />` (详情页系统通知区)
