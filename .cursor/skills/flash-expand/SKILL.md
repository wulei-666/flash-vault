---
name: flash-expand
description: 把一条闪念（只言片语或文章/视频 URL）判簇并展开成 Brief。用户丢闪念、说又一条闪念、要并入/拆开/升级为产品研判、或要看待决定时使用。
---

# 展开闪念

完整规则在仓库根目录 `AGENTS.md`。本 Skill 只给执行顺序。

## 步骤

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
