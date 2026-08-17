---
id: brief_vibehub-term-site-skill
cluster_id: vibehub-term-site-skill
title: VibeHub 术语站持续进化，Skill 来了
mission: LearnLand
status: ready
seed_ids:
  - seed_20260817_001
card_id: cards/vibehub-term-site-skill.md
related: []
source_gaps:
  - 未取得小红书笔记正文/转写（短链 404；discovery/item 报访问链接异常）
updated_at: 2026-08-17T14:10:00+08:00
---

# VibeHub 术语站持续进化，Skill 来了

## 这是什么

小红书分享标题指向 **VibeHub**（[vibe-hub.org](https://vibe-hub.org/)）：一个给 Vibe Coding 用的可视化术语图鉴。站点自我描述为 267 条词条；随后补了可安装的 **VibeHub Skill**（[vibe-hub.org/skill](https://vibe-hub.org/skill)、仓库 [oil-oil/vibe-hub-skill](https://github.com/oil-oil/vibe-hub-skill)），让 Agent 在写代码的同时把口语改成准确术语，并链回词条页。

## 主任务

LearnLand。用户原文只有分享标题和链接，没有「能不能做 / 拿去卖」；正文主题是学会用术语站和 Skill。文末留短产品信号。

## 和你的关系

`profile.yaml` 写明：前端、Vue、电商中后台；「名称叫法不一致会同时打沟通和 AI」。VibeHub 要解决的就是这件事：心里有界面，嘴里没有控件名，Agent 只能猜。

当前项目 `kayou.ecshopx.admin` 是电商后台。站点抽屉词条的示范句直接是「订单列表旁打开编辑收货地址的抽屉」——与中后台列表+详情的日常说法同构。本仓库自己也在用 Skill（`.cursor/skills/flash-expand`）：同一套 `SKILL.md` 机制。

Compete 不在这里用当前项目过滤独立 idea。通用 UI 术语站已经有 VibeHub；组内仍缺的是业务词（订单状态、券、售后），见文末产品信号。

## 待你决定

1. **今天做**：把官网安装口令发给当前 Agent，装上 `skills/vibehub`，拿一条真实的后台口语需求试改写。
2. **归档**：只当参考，不装 Skill、不跟站点。
3. **升级为产品研判**：不做第二个通用 UI 图鉴；若要产品化，单独开 Compete，题目改成「电商后台业务词表 + Skill」。

## 探索地图

```
口语需求 ──► VibeHub 词条（示例/边界/可复制提示词）
     │
     └──► VibeHub Skill（改写 + 每轮点名术语 + 解析器回链）
              │
              ├── 机制：Agent Skills / SKILL.md / 渐进式披露
              ├── 相邻：MCP（连外部系统）· Sub-agent
              └── 组内下一步：业务词 Skill（订单/券/售后叫法）
```

## 纵向：站点在教什么

### 1. 术语站本身

首页 JSON-LD（抓取自 [vibe-hub.org](https://vibe-hub.org/)）：

> 用界面示例、使用场景与边界讲清前端、后端、技术栈、AI、Agent、Git 与设计概念。用大白话也能搜到准确名称，更清楚向 AI 表达需求。共 267 个条目。

分类计数：Frontend 131、Backend 39、Product 11、Tech Stack 26、AI 24、Git 12、Design Styles 24，合计 267。

作者说明（[技术栈转载](https://jishuzhan.net/article/2080970743797129217)，对应掘金 7665700089651150882）：跨专业时「知道自己想做什么，却不知道该怎么准确说出来」。后端同学会说「这里再高级一点」「交互更顺滑一些」，说不清要的是卡片网格、抽屉、悬浮导航、骨架屏还是页面转场。词条之间左右切换；不要求坐下来系统学。

**词条体例（抽屉实证）**：[vibe-hub.org/drawer](https://vibe-hub.org/drawer)

| 块 | 实际内容 |
|---|---|
| 你可能会说 | 「从右边滑出来一块面板看详情，背后的列表还能看见。」 |
| 定义 | 从屏幕边缘展开、保留当前页面上下文的面板 |
| 何时用 | 列表旁看详情、筛选项多、稍长二级表单、移动端侧栏 |
| 何时不用 | 危险确认 → Modal；短内容 → Popover/Tooltip；抽屉套抽屉；多步长流程 → 新页 + Steps |
| 可复制提示词 | 订单列表右侧打开编辑收货地址的抽屉，保留背景与滚动位置；验证焦点、关闭返回、小屏 |

对电商后台：下次不要说「右边滑出来看看详情」，直接说 Drawer，并写清「保留列表、保存反馈、小屏布局」。

### 2. Skill 这个词在站点里指什么

[vibe-hub.org/skill](https://vibe-hub.org/skill) 原文：

> Skill 是一个有固定格式的文件夹，入口通常是 SKILL.md。文件夹里可以继续放说明、脚本、资料和模板。Agent 遇到对应任务时，会读取这些文件并按里面的做法完成工作。

页面给的对照实验：同一任务「把 12 条项目记录整理成周报」，只给原始记录时「结果容易变化」；同时使用周报 Skill 时「结果更容易复现」。

这与 Anthropic 文档一致。 [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) 写明：Skills 是可复用的文件系统资源；Prompt 是单次对话指令；Skills 按需加载，不必每次重复同一套指导。

### 3. 渐进式披露（机制，不是口号）

同一份 Anthropic 文档给出三级加载：

| 级 | 何时进上下文 | 代价 | 内容 |
|---|---|---|---|
| 1 Metadata | 启动时 | 约 100 tokens / Skill | YAML 里的 `name` + `description` |
| 2 Instructions | 任务匹配 description 之后 | SKILL.md 正文 | 步骤与约束 |
| 3+ Resources | 被引用时 | 未读则 0 | `references/`、脚本（脚本只把**输出**送进上下文） |

开放标准 [agentskills.io](https://agentskills.io/home) 把流程写成 Discovery → Activation → Execution。

VibeHub 自己的 Skill 就是这个结构：`SKILL.md` + `scripts/vibehub.mjs` + `vibehub.config.json`。术语正文不塞进 Skill 仓库，由网站维护——README 原句：「术语内容由 VibeHub 网站维护；这个仓库只保留表达规则和术语解析器。」

### 4. VibeHub Skill 现行行为（以仓库为准）

安装口令（官网与 README 相同）：

```text
https://github.com/oil-oil/vibe-hub-skill

帮我安装这个仓库里的 skills/vibehub Skill。
```

另一种安装命令（[skills.sh 条目](https://www.skills.sh/oil-oil/vibe-hub-skill/vibehub)）：

```text
npx skills add https://github.com/oil-oil/vibe-hub-skill --skill vibehub
```

现行 [SKILL.md](https://raw.githubusercontent.com/oil-oil/vibe-hub-skill/main/skills/vibehub/SKILL.md) / [README](https://raw.githubusercontent.com/oil-oil/vibe-hub-skill/main/README.md) 规定：

1. **改写**：口语 → 可直接发给 Agent 的术语句。示例：`这个按钮鼠标放上去变个色，点下去也要有一下反馈。` → `为按钮补充 Hover 状态和 Active 状态，并使用 Transition 让状态变化更自然。`
2. **每轮点名**：即使用户已经在改代码，追加「鼠标放到这个下载按钮上面时，加一个小提示」，也要在完成任务的同时标出 Tooltip，并链到 `https://vibe-hub.org/tooltip`。
3. **解析器**：先推断 1–3 个候选，再 `node scripts/vibehub.mjs resolve --query ... --compact`；禁止把用户整句当搜索词；只使用解析器返回的 url。
4. **不做的事**：不生成学习路线、课程、练习或术语清单；不擅自加框架/组件库/参数。

作者上线文写过「本地练习框架」。**现行 README 第一段写明不提供本地练习。** 以仓库现行文件为准。

隐私（README）：查询只发脱敏短描述；密钥、网址、邮箱、本地路径、代码块发送前移除。Anthropic 同一份 overview 的安全节写明：从外部 URL 拉内容的 Skill 有额外风险。该 Skill 会请求 vibe-hub.org。装之前读完 `SKILL.md` 和 `scripts/vibehub.mjs`。

Claude Code 的个人/项目 Skill 目录在官方文档里是 `~/.claude/skills/` 与 `.claude/skills/`。Cursor 侧对应本仓库已有的 `.cursor/skills/`。不同 Agent 路径不同，官网要求「安装时让当前 Agent 自己确认」。

## 横向：相邻概念与现成对照物

### Skill vs Prompt vs MCP vs 系统提示词

| | 解决什么 | 证据 |
|---|---|---|
| Prompt | 这一轮怎么说 | Anthropic：conversation-level, one-off |
| Skill | 这类任务的固定做法 + 可选脚本 | 站点定义 + SKILL.md 标准 |
| MCP | 连工单/数据库等外部系统 | [vibe-hub.org/mcp](https://vibe-hub.org/mcp)：协议不代替真实系统或授权 |
| 系统提示 / AGENTS.md | 这个仓库长期怎么干活 | 本仓库 `AGENTS.md`；站点 Skill 页「先知道」里把 System Prompt 列为相关概念 |

站点把 Skill 夹在 MCP 与 Sub-agent 之间：要连外部数据走 MCP；要把做法固化走 Skill；要并行拆任务走 Sub-agent。

### 不是同一件事的「术语站 / Skill」

| 对象 | 实际是什么 |
|---|---|
| VibeHub | 可视化词条 + 解析 Skill，面向「怎么跟 AI 把界面说准」 |
| [鱼皮 70 Vibe Coding 概念大全](https://www.codefather.cn/course/1935993640975368194/section/2010974737915121665) | 文章体词典（Token、上下文窗口、Agent Skills 定义） |
| [菜鸟教程 Vibe Coding](https://www.runoob.com/vibe-coding/vibe-coding-tutorial.html) | 教程站，进阶才提到 Skills / Workflow / Agent |
| 小红书 RED Skill | 社区里把 Skill 挂在笔记下分发（[36氪](https://36kr.com/p/3853737269892104) 等报道），不是术语图鉴 |

Vue 实现细节（如 `keep-alive`）不在本次抽到的 VibeHub 词条里（`/keep-alive` 返回 404）。框架 API 仍要查 Vue 文档；VibeHub 覆盖的是界面与协作用语。

## 明天能做

1. **装 Skill，用一条真需求验收。** 把第 4 节口令发给当前 Agent。用电商后台里真实出现过的口语，例如「订单行点开，右边滑出来改地址，列表别没了」。对照抽屉词条：应出现 Drawer，并保留列表与滚动位置。改写结果保存到你们的需求/工单里，给后端和测试同一套词。
2. **把组内会打沟通的叫法收进本仓库 Skill。** 本闪念仓库已经用 `.cursor/skills/flash-expand/SKILL.md`。后台项目同样可以建一个项目级 Skill：只写 kayou 后台里「列表 / 抽屉 / 空状态 / 二次确认」和业务词（订单状态、券、售后）的对照，触发条件写进 `description`。这是 Anthropic 说的 Custom Skills，不是再做一个 VibeHub。
3. **评审和产品对稿时打开一个词条当边界清单。** 抽屉页的「何时不用」直接挡住「用抽屉做删除确认」「抽屉里再开抽屉」。先用于当前迭代里最容易被说成「高级一点 / 顺滑一点」的一块界面。

## 连线

仓库里还没有其它 Brief。本簇与空队列无既有连线。

工作内可点名：本仓库的 `flash-expand` Skill（捕获闪念的固定流程）和 VibeHub Skill（捕获界面用语）是同一类资产，只是词表不同。

## 产品信号

通用「Vibe Coding 术语图鉴 + 安装即用的表达 Skill」已经有 VibeHub：267 词条、公开 Skill 仓库、MIT。再做一个前端组件图鉴，面对的是这个站点，不是空白市场。

仍空着的是 **垂直业务词表**：电商中后台里订单状态、优惠券、售后、结算字段的人话 ↔ 接口字段 ↔ 对 AI 的说法。那是另一份 Compete，需要你明确升级才会开。
