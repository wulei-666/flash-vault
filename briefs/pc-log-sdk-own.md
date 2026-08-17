---
id: brief_pc-log-sdk-own
cluster_id: pc-log-sdk-own
title: 自有生产级 PC SDK：文章方案 vs kayou_tracker
mission: Mixed
status: ready
seed_ids:
  - seed_20260817_002
card_id: cards/pc-log-sdk-own.md
related: []
source_gaps:
  - 本环境读不到 /Users/arvin/ky/kayou_tracker（路径不存在）；GitHub 账号 wulei-666 公开仓库列表中无 kayou_tracker。对照表已列出，缺本地 SDK 源码无法打分。
updated_at: 2026-08-17T14:40:00+08:00
---

# 自有生产级 PC SDK：文章方案 vs kayou_tracker

## 这是什么

你丢来一篇微信技术文，并写明：本地 `/Users/arvin/ky/kayou_tracker` 里也有一套 PC SDK，要**对照分析**，再**搞一套属于自己的、能在生产使用的 PC SDK**。

抓到的正文标题是《前端监控体系设计：监什么、怎么监、怎么落地（面试收藏级）》（公众号 Coding沉思录，2026-08-10）。文末写明覆盖 **Web / H5 / 小程序**，源码 [lotosv2010/g-heal-claw](https://github.com/lotosv2010/g-heal-claw)。采集手段是 `PerformanceObserver`、`sendBeacon`、IndexedDB，不是 Windows 原生进程日志库。

## 主任务

Mixed。学习信号：对照现有 SDK、搞懂方案。产品信号：我想搞一套属于自己的、能在生产使用。LearnLand 写落地对照；Compete 单独成章，不藏。

## 和你的关系

LearnLand 对照 `profile.yaml`：你是前端，栈是 Vue + 电商中后台 `kayou.ecshopx.admin`。文章里的 Web Vitals、白屏、XHR/fetch 拦截、SPA `pushState` 劫持，直接对应浏览器里的中后台，而不是 Electron/Win32 进程。

Compete 独立于当前后台项目：你要的是「自己的生产级 PC SDK」。禁止用「我们组做的是 Vue 后台」否定这套 idea。本地 `kayou_tracker` 才是对照物；本 Cloud Agent 读不到该路径。

## 待你决定

1. **今天做**：把 `kayou_tracker` 接到本会话（本机开权限、贴关键目录、或 push 到 GitHub），按文中对照表给现有 PC SDK 打分。
2. **归档**：只当学习笔记，不自研 SDK。
3. **先定边界**：目标是原生 Windows/C++ 客户端 SDK，还是 PC 浏览器（Web）监控 SDK。文章写的是后者；你的「PC SDK」用语指向前者。两条链路的竞品和落地清单不同，必须先选一条。

## 探索地图

```
微信文（Web 监控 SDK：Hub+Plugin / 四级上报 / 采样去重 / 隐私）
        │
        ├── 可迁移到原生 PC：队列、持久化、采样、幂等、beforeSend、体积预算
        │     不可照搬：PerformanceObserver、sendBeacon、IndexedDB、SPA 路由
        │
        ├── kayou_tracker（缺口：本环境无源码）
        │
        └── 自有生产 SDK
              ├── 若 Web：Sentry Browser / 阿里 ARMS / Fundebug / Webfunny / g-heal-claw
              └── 若原生 PC：Sentry Native+Crashpad / Bugly Windows(xlog) / 云厂商 C Producer
```

## LearnLand：文章在教什么

### 监什么（四个维度）

正文原句：「前端监控不是装一个 SDK 就完事了。」三个问题：监什么、怎么监、怎么落地。覆盖：**性能 + 异常 + 行为 + 自定义**。

| 维度 | 文中做法 | 生产口径 |
|---|---|---|
| 性能 | Core Web Vitals（LCP/FCP/CLS/INP/TTFB）+ NavigationTiming 九段 + 长任务三级（50ms / 2s / 5s） | LCP/INP/CLS 在 `pagehide` 或 `visibilitychange=hidden` 才封板；推荐 `web-vitals` 库 |
| 异常 | `onerror`、`unhandledrejection`、捕获阶段资源错误、白屏（根节点高度/子节点为 0） | 漏一类就有盲区 |
| 行为 | API 拦截、资源 `PerformanceObserver('resource')`、代码埋点 / 全埋点 / 曝光（停留 ≥500ms） | 核心漏斗用代码埋点 |
| 访问 | PV/UV、Session 30 分钟、劫持 `pushState/replaceState/popstate/hashchange` | SPA 漏劫持则 PV 偏低 |
| 自定义 | `track` / `time` / `log` 三接口，对应三张表 | `time` 覆盖 Web Vitals 管不到的业务耗时 |

文中点名的生产坑：SPA 路由切换后 Observer 不自动重置；`onCLS` 是最大窗口不是累积总量。

### 怎么监：Hub + Plugin

正文：「直接把所有采集逻辑堆在一个文件里，是大多数自研 SDK 最终变成屎山的原因。」

- Plugin：`setup(hub)` / 可选 `teardown`
- Hub：DSN、插件加载、全局队列、`report(event)`
- 七个内置插件：Error / Performance / Api / Resource / PageView / WhiteScreen / Exposure
- `teardown` 用于 SPA 与微前端子应用卸载，否则重复上报
- 初始化第三步：SDK 早于业务时，`track/log` 先入队再 flush，否则静默丢失
- **体积硬指标**：核心+错误+性能 gzip **< 15KB**，全量 **< 30KB**；宿主长任务占比 **≤ 2%**

### 上报：四级降级 + 离线队列

1. `navigator.sendBeacon`（`pagehide`/`beforeunload`，单次 ≤ 64KB）
2. `fetch({ keepalive: true })`
3. XHR
4. `new Image().src`（单条 ≤ 2KB）

超 64KB：error / session_end 走 Beacon，其余进 IndexedDB。失败队列上限 **500 条**，超量丢最旧；启动和 `online` 时重试。

文中对照：医疗问卷 SPA 上，`beforeunload + fetch` 关闭 Tab 丢失率约 30%；`pagehide + sendBeacon` 后接近 0。

### 落地三关

| 关 | 文中规定 |
|---|---|
| 采样 | 客户端：error/track=1.0，performance/api=0.3；服务端再滤一层保库 |
| 去重 | SDK `eventId` 用 UUID v7，重试复用；服务端 UNIQUE + `ON CONFLICT DO NOTHING` |
| 隐私 | `beforeSend`；password/token/authorization/cookie/secret → `[FILTERED]`；email/name 不入库；body 截断 4KB |

告警用相对基线而不是死绝对值。文末「智愈」用 LangChain ReAct + Docker 沙箱 + 自动 PR；安全边界：`heal.paths` 白名单、LOC 上限、沙箱不联网。这是平台能力，不是 SDK 最小闭环。

### 和 kayou_tracker 怎么比（源码未到）

本环境：`/Users/arvin/ky/kayou_tracker` 不存在。`gh repo list wulei-666` 无同名仓库。下面是对照清单，源码到了按「有/无/部分」填，禁止空口打分。

| 检查项 | 文章（Web SDK）里的标准 | 原生 PC SDK 应对等物 | kayou_tracker |
|---|---|---|---|
| 采集与上报分离 | Hub 管队列，Plugin 管采集 | Core + 崩溃后端 / 日志后端拆开 | 待填 |
| 插件 teardown | 防重复监听 | DLL 卸载、进程退出 `flush`/`sentry_close` | 待填 |
| 早于业务的调用 | 初始化前 API 入队 | 初始化前 log 丢弃或环形缓冲 | 待填 |
| 进程崩溃仍能送出 | Beacon 卸载发送 | 进程外 handler（Crashpad）或下次启动补传 | 待填 |
| 本地持久化 | IndexedDB 500 条 | mmap/xlog/SQLite，目录必须可写 | 待填 |
| 采样 | 错误全量、性能抽样 | 崩溃全量、info 日志抽样 | 待填 |
| 幂等 | UUID v7 | 崩溃报告 id / 日志 batch id | 待填 |
| 隐私 | beforeSend | 默认不采剪贴板/路径里的密钥 | 待填 |
| 体积/开销预算 | gzip <15KB / 长任务 ≤2% | 初始化耗时、工作线程、磁盘 | 待填 |
| 生产限制 | Beacon 64KB | Bugly：打包前 >500MB 放弃；5 分钟最多 2 次；日限额 500MB | 待填 |

不可照搬到 Win32/C++ 的 API：`PerformanceObserver`、`sendBeacon`、IndexedDB、`history.pushState`。可照搬的是**分层、队列、采样、幂等、隐私钩子、体积预算**。

### 明天能做（LearnLand，基于 profile）

1. **先定端**。若你的 PC SDK 是卡游客户端（原生），不要把这篇文章当实现说明书，只当生产清单。若你其实要给 `kayou.ecshopx.admin` 做浏览器监控，文章可直接当架构底稿。
2. **源码一到就填表**。打开 `kayou_tracker` 搜：初始化、flush、崩溃 handler、日志目录、上传、采样、脱敏。缺哪行补哪行。
3. **后台项目只做一件对照实验**（Web 路径）：在 Vue 后台对「接口失败」走 error 全量、「路由切换 PV」走 `pushState` 劫持。这验证文章，不替代原生 PC SDK。

## Compete：自有生产级 PC SDK

独立于当前电商后台。题目是你自己的话：「搞一套属于自己的，能在生产使用的 PC 的 sdk」。

### 先拆产品定义

市场上「PC SDK」至少两摊，不能混成一个仓库说明书：

| | A. PC 浏览器监控 SDK | B. 原生桌面（Windows/macOS）日志/崩溃 SDK |
|---|---|---|
| 宿主 | Chrome/Edge 里的网页、中后台 | exe / dll / 游戏客户端 |
| 本文 | 正是这篇 | 不是这篇 |
| 你的 kayou_tracker | 待源码确认 | 待源码确认 |

选 A：竞品是浏览器监控。选 B：竞品是 Crashpad/xlog。两边都做，等于两套 SDK。

### 有没有人做（必须列出的竞品）

**A. Web / H5 监控**

| 产品 | 证据 | 和「自研」的差 |
|---|---|---|
| Sentry Browser | [docs.sentry.io](https://docs.sentry.io/) 浏览器 SDK；Hub+Integration 是事实标准 | 功能全，数据出域，体积与定价 |
| 文章作者的 GHealClaw | [github.com/lotosv2010/g-heal-claw](https://github.com/lotosv2010/g-heal-claw)，3 star；采集 Web/H5/小程序 | 面试级全貌 + AI 自愈，不是桌面客户端 |
| 阿里云 ARMS / Quick Tracking Windows | [Windows C++ 基础集成](https://help.aliyun.com/zh/document_detail/2865670.html)：`initQTPC`，隐私政策授权后才能 init；默认 300 条聚合、间隔 3s | 埋点增长，不是崩溃 minidump |
| Fundebug / Webfunny / mitojs | 国内监控 SaaS 与开源采集层 | 卖的是平台，不只是 dll |

**B. 原生 PC 日志 / 崩溃**

| 产品 | 证据 | 生产约束（文档原句级） |
|---|---|---|
| Sentry Native + Crashpad | [Native 文档](https://docs.sentry.io/platforms/native/)：Windows/macOS/Linux；**必须随包分发 `crashpad_handler.exe`**；退出前 **`sentry_close()`** 否则事件丢失；database 生产环境不要用相对 CWD，应写 `%AppData%\Local` | 文档写明 standalone 使用仍是 experimental，主用途是给其他 SDK 供血 |
| Bugly Windows 日志 | [bugly.tds.qq.com windows 日志](https://bugly.tds.qq.com/docs/sdk/windows_diagnose/)：`bugly_logger.dll` = diagnose（捞取/染色/条件采集）+ Logger（底层微信 **xlog**）；业务可自带 logger，只把路径交给 diagnose | 主动上报：压缩前 >**500MB 放弃**；**5 分钟最多 2 次**；每天压缩后 **500MB** |
| 阿里云日志 C Producer | [aliyun-log-c-sdk](https://github.com/aliyun/aliyun-log-c-sdk)：纯 C、批量、lz4、发送线程 | 采集进 SLS，不管崩溃堆栈 |
| 腾讯云 CLS C SDK | [C SDK 上传日志](https://www.tencentcloud.com/zh/document/product/614/74219)：`ProducerConfig`、批量、压缩 | 同上，是日志管道 |
| 美团 Logan | 移动端 C 库 + mmap + 压缩加密；Web 走 IndexedDB | 开源主战场不是 Win32 客户端 |

结论（有证据、无空话）：**生产级 PC 端能力已经被 Sentry Native / Bugly / 云厂商 Producer 占住。** 空白不在「再写一个会 fwrite 的 logger」，而在 **和 kayou 客户端绑定的：账号/设备/业务事件 + 可捞日志 + 崩溃 minidump + 自有存储**。

### 值不值得自研

自研成立的条件（同时满足才做，缺一就买/嵌现成后端）：

1. 数据必须进自有数仓，Sentry/Bugly 出域不可接受。
2. 现有 `kayou_tracker` 已经有宿主集成面（初始化、userId、通道），缺的是生产可靠性而不是从零 API。
3. 你能养：**崩溃符号仓库、捞日志通道、采样配置下发、隐私审查**。文章把 AI 自愈写成终极形态；那是第二年的平台，不是 SDK v1。

不成立：只想「有个自己品牌的 dll」，功能对标 Sentry Native。那是重复造轮子，文档已经写了 handler 分发、WER/fast-fail、可写 database 路径这些坑。

### 落地（v1 生产清单，不写公司明天排期）

按文章的「体积是第一约束」迁到桌面端，v1 只做四件事：

1. **Core**：init / shutdown / 环形或 mmap 队列 / `eventId` / `beforeSend`。
2. **Crash 后端**：进程外 handler（对齐 Crashpad），或明确「仅下次启动上传」（对齐 Breakpad/inproc）。Windows 要处理 WER fast-fail：Sentry 文档写明这类崩溃**不会**走 `before_send`。
3. **Log 后端**：可插拔。自研文件格式或接 xlog；目录可写；进程异常必须 flush（Bugly 示例在结束/异常时 `flushLog`）。
4. **上传策略**：错误/崩溃全量；info 抽样；失败落盘；有日配额（对齐 Bugly 500MB/天，避免打爆用户磁盘和你的入口）。

Web 文章里的 Beacon 四级降级，在原生侧对应：**崩溃当下由 handler 发送 vs 下次启动补传**。选一个写进 README，不要两种混用还不说明。

体积/开销预算要写进验收，对应文章 gzip 15KB / 长任务 2%：例如 init < N ms、工作线程 ≤1、默认日志目录不进安装目录（CWD 不可写，Sentry 已警告）。

### 发散

- 只做采集 SDK，后端用 CLS/SLS，自己不养大盘。
- 只做崩溃，日志继续用现有 `kayou_tracker`。
- 只做 Web 监控给中后台，原生 PC 继续嵌 Bugly/Sentry。

三条都能叫「自己的 SDK」，范围差一个数量级。待你决定第 3 条就是锁范围。

## 连线

仓库内无其它存活 Brief。已删除的 VibeHub 簇与本簇无关。

## 产品信号

文章给的是 **Web 监控 SDK 的生产课纲**，不是现成的 Win32 日志库。自研 PC SDK 的差异化在 kayou 业务上下文和数据不出域，不在再实现一套 `PerformanceObserver`。源码对照完成前，不要开第二份「Web 监控」Brief。
