# flash-vault Agent 指令

你是闪念操作系统的工人。正本是本仓库的文件，不是聊天记录。

电脑可能关机。你跑在 Cloud Agent 里：从 GitHub 克隆本仓库，写文件后 commit 并 push / 开 PR。

## 每次必须做的两件事

1. **改仓库**：写入或更新 Seed / Card / Brief / `queue.md`。
2. **在对话里先回 Brief 头**（给手机看，30 秒能扫完）。不要只写文件不回对话，也不要只聊天不写文件。

Brief 头格式固定：

```
【这是什么】一句话
【主任务】LearnLand | Compete | Mixed
【本次】新建 Brief `briefs/xxx.md` 或 并入已有 Brief `briefs/xxx.md`（写明原因）
【待你决定】
1. …
2. …
3. …
【缺口】无 | 未取得正文/转写（说明缺什么）
```

用户用一句话做决定时（今天做 / 归档 / 升级为产品研判 / 并到某份 / 拆开），更新对应 Brief 与 `queue.md`，再回一版新的 Brief 头。

## 捕获

用户发来的每一条都是 Seed，三种平等，都要展开：

- 只言片语
- 文章 URL（无评也要抓正文、爆炸）
- 视频 URL（无评也要转写后爆炸）

禁止把链接当成「附件」。禁止因为用户没写评就跳过。

## 判簇（先判再写 Brief）

处理新 Seed 之前，读取 `cards/`、`briefs/`、`queue.md`、`seeds/`。

- **同一簇**：相同或高度类似的一个闪念（同一规范化 URL、同一主题、连发多条围同一件事、隔天补一句仍是那件事）。  
  → 不新开 Brief。把新 Seed 挂到旧簇，**更新那一份 Brief**（汇总、扩散、改结论）。禁止再出一份类似报告。
- **相关但不是同一件事**：新 Brief，双向写入双方的 `related`，LearnLand 写「明天能做」时允许点名另一份。
- **无关**：新 Brief。

用户可纠正：说「拆成两条」或「并到某份」时改文件，不要争辩。

## 路由

看用户原文；只有 URL 时看抓到的正文。

| 主任务 | 信号 | 展开 |
|---|---|---|
| LearnLand | 学会、怎么用、借鉴、用到工作/团队、排障 | 全文学习 + 基于 profile **和** 已连线闪念写「明天能做」；文末短「产品信号」；连线 |
| Compete | 能不能做、有没有人做、我想做个、产品化、变现、拿去卖 | 发散 / 价值 / 落地 / **必须搜索竞品**；不把「明天在公司干什么」当主结论；连线 |
| Mixed | 两种信号都有 | LearnLand 为主落地；Compete 单独成章，写完，不藏 |

封版例子：

- 「列表滚动白屏，怀疑是 keep-alive。」→ LearnLand
- 「能不能做一个把会议纪要自动变成接口字段对照表的工具，拿去卖。」→ Compete
- 「这个术语站很好，我们组也能用，说不定也能单独做成产品。」→ Mixed

`profile.yaml` 只给 LearnLand 写「能否用到当前工作 / 明天做什么」。  
**禁止**用当前项目否定一个独立 idea。Compete 不读「当前项目」来过滤想法。

没有连线时，LearnLand 只靠本簇 + profile。

## 爆炸

- **纵向**：文中每个技术点是什么、怎么用、挖到能落地。
- **横向**：相邻概念、对比方案、和已有闪念 / 当前栈的关系。

文章：抓正文。抓不到：Brief 仍写，标明「未取得正文」，基于标题、页面可见文字、连线闪念展开，把「补材料」放进待你决定。禁止静默跳过。

视频：先转写再爆炸。没有口播正文（如部分小红书）：仍写 Brief，标明缺口，用标题、页面文案、用户的话、连线展开。禁止把「先看完视频」当成展开前提。

每条 Seed 都要处理。用「一簇一份 Brief」去重，不要用条数上限丢弃。

## 文件怎么写

- 新 Seed → `seeds/YYYY-MM-DD-HHMM-<短slug>.md`（原文原样保留，禁止改写用户的话）
- 抓到的正文/转写 → `sources/`，Seed frontmatter 里指向它
- 一簇一张主卡 → `cards/<cluster-id>.md`（判断句尽量用用户自己的话）
- 一簇一份 Brief → `briefs/<cluster-id>.md`
- 更新 `queue.md`：`status: ready` 的 Brief 必须出现在待决定；已归档的移到「已决定」

frontmatter 字段见各目录下的 `_schema.md`。新建 Brief 必须含：这是什么、主任务、待你决定、探索地图，再写对应章节。

## Git

改完后 commit，信息用中文，写清：新建还是并入、主任务、Seed 短描述。  
Cloud Agent：push 分支或开 PR。  
本地 Agent：commit，是否 push 听用户。

## 不要做的

- 不把闪念写进其他业务仓库
- 不做知识图谱 UI
- 不把对话记录当正本
- 不要求用户捕获时分类、打标题、排版

## Cursor Cloud specific instructions

本仓库是**纯内容 + Agent 工作流**仓库，不是传统可编译应用：没有依赖清单、构建系统、lint、测试框架、`.cursor/environment.json` 或 Dockerfile。因此：

- **无需安装依赖**。启动更新脚本只做 `git --version` 之类的健康检查即可，别加 `npm install` / `pip install` 这类步骤（没有可装的东西）。
- **没有 lint / test / build / dev server 命令**。「运行应用」= 执行上文的闪念展开工作流：捕获 Seed → 判簇 → 路由 → 写 `briefs/` + `cards/` → 更新 `queue.md` → commit/push。验证环境是否可用 = 能否把一条闪念端到端处理成 Brief。
- **运行期只依赖三件事**：`git`（写文件 + commit/push）、文件读写、以及抓取文章正文/视频转写的联网能力（Cloud VM 出网正常，`example.com` 抓取已验证通过）。抓不到正文时按 `AGENTS.md` 在 Seed/Brief 里标缺口，别静默跳过。
- Cloud Agent 改完必须 commit 并 push / 开 PR（正本在 GitHub，不在本机磁盘）。
- `env-check-*` 前缀的簇是环境自检 Demo，与真实闪念无关，可随时整簇删除。
