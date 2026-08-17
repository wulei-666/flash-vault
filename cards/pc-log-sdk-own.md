---
id: pc-log-sdk-own
cluster_id: pc-log-sdk-own
claim: 是 PC 浏览器（Web）监控 SDK；对照 kayou_tracker 后，把现有 web SDK 补到生产清单，不从零再写 Hub
brief: briefs/pc-log-sdk-own.md
related: []
updated_at: 2026-08-17T15:55:00+08:00
---

边界已锁为 Web 监控。kayou_tracker 的 web/ 已有 Tracker+插件+logcenter；缺采样、队列上限、pagehide、脱敏、CLS/INP、白屏、eventId、Observer teardown。
