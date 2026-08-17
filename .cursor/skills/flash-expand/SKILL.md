---
name: flash-expand
description: 处理闪念或同一会话里的维护指令。用户丢闪念、又一条闪念、做决定、删除闪念、或改 AGENTS 工作流时使用。
---

# 展开闪念 / 维护仓库

完整规则在仓库根目录 `AGENTS.md`。先做消息分类，禁止把「删掉」「改规则」「今天做」写成新 Seed。

## 分类

- **闪念** → 下面「展开步骤」
- **决定 / 删除 / 改工作流** → 按 `AGENTS.md` 改已有文件，commit，对话说明改了什么。决定类回 Brief 头。

拿不准先问，不要新建 Brief。

## 展开步骤（仅闪念）

1. 把用户消息存成 `seeds/` 新文件，原文不改写。有 URL 则 `kind` 含 `article_url` 或 `video_url`。
2. 能抓则抓正文/转写，写入 `sources/`；失败则在 Seed 和 Brief 上标明缺口，继续。
3. 对照 `cards/`、`briefs/`、规范化 URL、语义主题，判簇：同一簇 | 相关 | 无关。
4. 定主任务：LearnLand / Compete / Mixed（规则与封版例子见 `AGENTS.md`）。
5. 同一簇：更新那份 Brief 和主 Card，把新 `seed_id` 写进 frontmatter。
6. 新簇：新建 `cards/<id>.md` 与 `briefs/<id>.md`。
7. 相关：新 Brief，双方 `related` 互写。
8. 更新 `queue.md`。
9. Git commit（Cloud Agent 再 push / 开 PR）。
10. 对话里只先回 Brief 头，不要把全文糊进聊天。用户说「看全文」再贴或指出文件路径。

## Brief 头

```
【这是什么】
【主任务】
【本次】新建 | 并入 `briefs/....md`
【待你决定】最多 3 条
【缺口】
```
