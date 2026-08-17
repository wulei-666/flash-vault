# VibeHub 术语站与 Skill — 抓取记录

抓取时间：2026-08-17（UTC）

## 1. 用户给的小红书短链 — 未取得笔记正文

- 用户 URL：`http://xhslink.cn/o/3WgZ9SMxnYx`
- HTTPS 同路径：`https://xhslink.cn/o/3WgZ9SMxnYx`
- 两次请求均返回 HTTP 404（nginx，`Content-Length: 18`）
- 把短码当成笔记 id 访问 `https://www.xiaohongshu.com/discovery/item/3WgZ9SMxnYx`：先 302 到 `website-login/error`，`error_code=300017`，错误文案「访问链接异常」

因此：**没有小红书笔记正文，也没有视频转写。** 下面材料来自笔记标题所指站点及其 Skill 仓库、作者公开说明、Agent Skills 官方文档。

## 2. 站点自我描述（vibe-hub.org 首页 JSON-LD）

来源：`https://vibe-hub.org/` 页面内 schema.org `WebSite` / `DefinedTermSet`

> VibeHub（vibe-hub.org）是面向所有 Vibe Coding 用户的可视化术语图鉴：用界面示例、使用场景与边界讲清前端、后端、技术栈、AI、Agent、Git 与设计概念。用大白话也能搜到准确名称，更清楚向 AI 表达需求。共 267 个条目。

- `numberOfItems`: 267
- `inLanguage`: zh-CN
- 英文镜像：`https://vibe-hub.org/en`
- 规范 URL：`https://vibe-hub.org`

首页分类计数（页面可见）：Frontend 131 / Backend 39 / Product 11 / Tech Stack 26 / AI 24 / Git 12 / Design Styles 24。

131+39+11+26+24+12+24 = 267，与 JSON-LD 一致。

## 3. Skill 词条页（vibe-hub.org/skill）

来源：`https://vibe-hub.org/skill`

站点对 Skill 的定义（摘录）：

> Skill 是一个有固定格式的文件夹，入口通常是 SKILL.md。
> 文件夹里可以继续放说明、脚本、资料和模板。Agent 遇到对应任务时，会读取这些文件并按里面的做法完成工作。

安装口令（页面给出的原文）：

```
https://github.com/oil-oil/vibe-hub-skill
帮我安装这个仓库里的 skills/vibehub Skill。
```

使用示例：

```
使用 VibeHub，帮我学懂 Git Branch。
```

页面写明：Skill 也可以来自 GitHub、SkillHub、别人发来的本地文件夹；不同 Agent 保存位置不同，安装时让当前 Agent 自己确认。

相邻词条导航：MCP ← Skill → Sub-agent。

页面对比实验（同一任务「把 12 条项目记录整理成周报」）：只给任务和原始记录时「结果容易变化」；同时使用周报 Skill 时「结果更容易复现」。

## 4. VibeHub Skill 仓库 README（现行规则）

来源：`https://raw.githubusercontent.com/oil-oil/vibe-hub-skill/main/README.md`

> VibeHub Skill 是一个面向普通人的 Vibe Coding 术语表达助手。它把不够准确的口语描述改成可以直接发给 Agent 的专业需求，也会主动指出当前场景里最值得认识的术语，附上通俗解释和 VibeHub 链接。
>
> 它不提供学习路线、课程计划或本地练习，也不会把一句简单需求扩展成教程。

改写示例：

- 输入：`这个按钮鼠标放上去变个色，点下去也要有一下反馈。`
- 输出：`为按钮补充 Hover 状态和 Active 状态，并使用 Transition 让状态变化更自然。`

仓库结构：

```
skills/vibehub/
├── SKILL.md
├── agents/openai.yaml
├── scripts/vibehub.mjs
└── vibehub.config.json
```

> 术语内容由 https://vibe-hub.org/ 网站维护；这个仓库只保留表达规则和术语解析器。

隐私：术语查询只发送脱敏后的简短描述；常见密钥、网址、邮箱、本地路径和代码块会在发送前移除。

许可：MIT。GitHub 页面显示 101 stars（抓取当日）。

`vibehub.config.json`：

```json
{
  "schemaVersion": 1,
  "siteUrl": "https://vibe-hub.org",
  "repositoryUrl": "https://github.com/oil-oil/vibe-hub-skill",
  "skillPath": "skills/vibehub"
}
```

## 5. 现行 SKILL.md 全文要点

来源：`https://raw.githubusercontent.com/oil-oil/vibe-hub-skill/main/skills/vibehub/SKILL.md`

frontmatter `name: vibehub`

核心指令：

1. 把模糊描述改成可以直接交给 Agent 的准确需求。
2. 主动指出用户刚刚描述但没有说出名称的技术概念。
3. **不要生成学习路线、课程、练习或术语清单。**
4. 每轮检查用户新增的话，不因已开始写代码就跳过术语检查。
5. 先完成开发任务，再把最相关的一个术语写进结果；第一次出现用解析器返回的 Markdown 内链。
6. 用 `scripts/vibehub.mjs resolve --query ... --compact` 批量验证候选术语；不要把用户整句原话直接当搜索词。
7. 只使用解析器返回的 `url`，不自行拼接或伪造链接。
8. 查询只传脱敏后的短术语。

## 6. 作者公开说明（与现行 Skill 有一处不一致）

来源：技术栈转载 `https://jishuzhan.net/article/2080970743797129217`（文内写明对应掘金 `https://juejin.cn/post/7665700089651150882`；掘金原文本次抓取超时）

作者自称：最近用 Vibe Coding 做了 vibe-hub.org；上线第一天 Google Analytics 与 Vercel Analytics 合计访问量 2k+。

要解决的问题（作者原话结构）：跨专业时「知道自己想做什么，却不知道该怎么准确说出来」。后端同学会说「这里再高级一点」「交互更顺滑一些」，说不清要的是卡片网格、抽屉、悬浮导航、骨架屏还是页面转场。

站点覆盖：前端组件、页面布局、视觉与交互、CSS 与网页基础、后端与网络、数据与账号、部署排错、Git 和设计风格。词条含界面示例、使用场景、常见边界；条目间可左右切换。

Skill 段落（作者文章）：

> Skill 可以让 Agent 拿到网站上的术语信息，也能拿到准确链接，在回复里内联，或者在内置浏览器里打开。另外还带了一个本地练习框架，Agent 可以参考术语和你的学习需求，定制小练习。

**与现行 README / SKILL.md 冲突：** 仓库现行文本明确「不提供学习路线、课程计划或本地练习」。展开时以仓库现行文件为准；把作者文章当作上线说明，不当作 Skill 现行行为。

## 7. 词条体例实证：抽屉 Drawer

来源：`https://vibe-hub.org/drawer`

口语入口：「从右边滑出来一块面板看详情，背后的列表还能看见。」

定义：从屏幕边缘展开、在保留当前页面上下文时显示内容的面板组件。

何时用 / 何时不用：列表旁看详情用抽屉；确认删除用 Modal；短内容用 Popover/Tooltip；不要抽屉套抽屉；多步骤长流程用新页面 + Steps。

「你可以这样告诉 AI Agent」示例直接是订单列表场景。

## 8. 相邻概念：MCP

来源：`https://vibe-hub.org/mcp`

> MCP 是 AI 应用发现和调用外部工具、资料与提示词时使用的一套连接协议。
> MCP 统一连接方式，但不代替真实系统或授权。

与 Skill 的站点关系：Skill 页左右邻接 MCP 与 Sub-agent。

## 9. Agent Skills 官方机制（Anthropic）

来源：`https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview`

- Skill 是文件系统上的资源：指令 + 元数据 + 可选脚本/模板。
- 与 Prompt 的差别：Prompt 是单次对话级；Skills 按需加载，不必在每次对话重复同一套指导。
- 渐进式披露三级：
  - Level 1 Metadata：启动时加载 `name` + `description`，约 100 tokens / Skill
  - Level 2 Instructions：触发后才读 SKILL.md 正文
  - Level 3+ Resources：按需读 references/scripts；脚本只把输出送进上下文
- Claude Code 自定义 Skill 路径：`~/.claude/skills/`（个人）或 `.claude/skills/`（项目）
- 安全：只使用可信来源；从外部 URL 拉内容的 Skill 有额外风险（拉取内容可含恶意指令）

开放标准首页：`https://agentskills.io/home` — Skill 核心是含 `SKILL.md` 的文件夹；Discovery → Activation → Execution。
