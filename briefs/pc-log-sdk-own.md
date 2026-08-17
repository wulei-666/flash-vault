---
id: brief_pc-log-sdk-own
cluster_id: pc-log-sdk-own
title: 自有生产级 Web 监控 SDK：补 kayou_tracker
mission: Mixed
status: ready
seed_ids:
  - seed_20260817_002
card_id: cards/pc-log-sdk-own.md
related: []
source_gaps:
  - Cloud Agent 仍不能克隆 Codeup `git@codeup.aliyun.com:kayou/qianduan/kayou_tracker.git`。对照结论来自补材料 sources/2026-08-17-kayou-tracker-compare.md（用户本机读源码），不是本环境克隆。
updated_at: 2026-08-17T15:55:00+08:00
---

# 自有生产级 Web 监控 SDK：补 kayou_tracker

## 这是什么

微信文《前端监控体系设计》对照卡游 `kayou_tracker` 的 Web SDK。边界已锁：**PC 浏览器（Web）监控 SDK**，不是 Win32 dll。现有实现在 `web/`（另有小程序 `web/src-mini/`、Flutter `app/`）。生产动作是把 tracker **补到文章的生产清单**，不是从零再写一个 Hub。

## 主任务

Mixed。学习：文章课纲 vs tracker 源码对照。产品：自己的、能上生产的 Web 监控 SDK——对照后结论是升级现有 SDK，不是新开仓库对标 Sentry Native。

## 和你的关系

`profile.yaml`：Vue 电商中后台 `kayou.ecshopx.admin`。宿主就是 PC 浏览器。`kayou_tracker` 根 README 三套：Web / 小程序 / Flutter。补材料写明默认上报卡游 `logcenter`（自有数仓）。

Compete 不拿「后台项目已经有 tracker」否定独立产品；本簇结论是 **Web 路径已有 Hub，缺生产件**。原生 Windows 已排除。

## 待你决定

1. **今天做**：按下面 7 项缺口改 `kayou_tracker` `web/`（采样、失败队列上限、pagehide、默认脱敏、CLS/INP+白屏、eventId、Observer teardown）。
2. **归档**：对照到此为止，不改 tracker。
3. **给 Cloud Agent 源码**：把 Codeup 镜像成 GitHub 私有仓并接入 multi-repo；不镜像则继续用贴表当补材料。

## 探索地图

```
微信文课纲（Hub+Plugin / 四级上报 / 采样 / UUID v7 / beforeSend / Vitals）
        │
        └── kayou_tracker web/  （已有 Tracker + 五插件 + logcenter）
                │
                ├── 已对齐：采集上报分离、sendBeacon 优先、filter/use 钩子
                ├── 缺口 7 项 → 生产清单
                └── 不做：Win32 / Crashpad / Bugly dll
```

## LearnLand：文章课纲（保留）

正文三个问题：监什么、怎么监、怎么落地。覆盖性能 + 异常 + 行为 + 自定义。

硬指标仍以文章为准：核心+错误+性能 gzip **< 15KB**，全量 **< 30KB**；错误全量、性能抽样 0.3；卸载走 **pagehide + sendBeacon**（文中问卷 SPA：`beforeunload+fetch` 丢失约 30%，改 pagehide 后接近 0）；失败队列 IndexedDB **500** 上限；`eventId` UUID v7；`beforeSend` 过滤 password/token。

七插件课纲：Error / Performance / Api / Resource / PageView / WhiteScreen / Exposure。

## LearnLand：kayou_tracker 对照（补材料已填）

证据：`sources/2026-08-17-kayou-tracker-compare.md`。用户标明读过 `web/src/core/Tracker.ts`、`ErrorPlugin.ts`、`PerformancePlugin.ts`、`web/README.md` TODO。远程 `git@codeup.aliyun.com:kayou/qianduan/kayou_tracker.git`。

| 检查项 | 文章标准 | kayou_tracker（Web） |
|---|---|---|
| 采集与上报分离 | Hub + Plugin | **有。** Tracker 管队列/上报；Performance / Behavior / Api / Error / Track 采集 |
| 插件 teardown | teardown 防重复监听 | **部分。** `Tracker.destroy()` → 插件 `dispose`；ErrorPlugin 有 `removeEventListener`。README TODO「销毁？」；PerformancePlugin **未见** dispose Observer |
| 早于业务的调用 | init 前 API 入队 | **无独立预队列。** `new Tracker()` 即注册并 `pushLog`。未开 Track 插件时 `track()` 直接 return |
| 卸载仍能送出 | pagehide + sendBeacon | **部分。** `sendLogs` 优先 sendBeacon，失败 XHR。绑在 `beforeunload` 和 `visibilitychange=hidden`，**不是** pagehide。无 fetch keepalive、无 Image 第四级 |
| 本地持久化 | IndexedDB，上限 500 | **localStorage** 键 `trackerFailedLogs`。失败重试 3 次再写入。代码 TODO 上限——**未做** |
| 采样 | error 全量、性能 0.3 | **无。** README TODO「采样率？」 |
| 幂等 | UUID v7 | **无 eventId。** sessionId / visitId = 时间戳+随机 |
| 隐私 | beforeSend | **部分。** 有 `filter` / `use`。README TODO 脱敏加密——**未做**默认密钥过滤 |
| 体积/开销 | gzip <15KB / 长任务 ≤2% | 有 `maxFieldLength`、超大 body 截断。**未见**包体/长任务验收 |
| 性能指标 | LCP/INP/CLS | Observer 收 **LCP/FCP/FP**、长任务、资源。**未见 INP/CLS** |
| 白屏 | 根节点高度/子节点 | **无** WhiteScreen 插件 |
| SPA PV | 劫持 pushState | BehaviorPlugin 有路由监听（含 pagehide）。Vue 后台是否劫持 history **表内未闭合** |
| 数据去向 | 自有上报 | 默认 **logcenter**，不出域到 Sentry |

Flutter：`path_provider` jsonl、约 2MB 轮转、冷启动/回前台重传；Native crash 自行丢 `tracker/native_error`。那是 App，已排除出本簇产品边界。

### 明天能做（Vue 中后台 + tracker）

1. **在 `kayou.ecshopx.admin` 核对 BehaviorPlugin 是否劫持 `history.pushState`。** 表内未闭合。SPA 后台若只靠 pagehide，路由切换 PV 会按文章口径偏低。
2. **先补三件不改产品形态的缺口：** 失败队列上限（localStorage 删最早）、卸载加 `pagehide`、默认把 password/token/authorization 从 `filter` 里挡住。对应 README 已有 TODO。
3. **采样与 eventId 一起做：** error=1.0、performance/api 可配；每条日志 UUID v7，重试复用；logcenter 侧 UNIQUE。没有这两项，全量性能日志和高重试会打满数仓。

## Compete：自有生产级 Web 监控 SDK

独立于「今天后台排期」：题目仍是你的话——属于自己的、能在生产使用。边界已定为 **PC 浏览器 Web SDK**。

### 产品定义（已锁）

| | 已选：A. PC 浏览器监控 SDK | 已排除：B. 原生 Win32 |
|---|---|---|
| 宿主 | Chrome/Edge 中后台、Vue | exe / dll |
| 对照物 | `kayou_tracker` 的 `web/` | tracker 里没有 |
| 微信文 | 同一条链路 | 不是 |

### 有没有人做

| 产品 | 证据 | 和本簇的差 |
|---|---|---|
| kayou_tracker `web/` | 补材料：Tracker + 五插件 + logcenter | **已有 Hub**，缺 7 项生产件 |
| Sentry Browser | Hub + Integration；数据出域 | 你已走自有 logcenter，替换成本是出域 |
| GHealClaw | [lotosv2010/g-heal-claw](https://github.com/lotosv2010/g-heal-claw)，Web/H5/小程序 + AI 自愈 | 课纲级全貌；3 star；不是卡游业务字段 |
| 阿里云 ARMS | 商业 APM | 数据与定价在对方 |
| Fundebug / Webfunny / mitojs | 国内 SaaS / 开源采集 | 卖平台或通用 SDK，不绑 logcenter |

原生 Sentry Native / Bugly Windows / 云厂商 C Producer：**本簇不再作为交付选项。** 需要时另开 Brief。

### 值不值得「再搞一套」

补材料原句：「在 Web 上更像是把 kayou_tracker 补到生产清单，不是从零再写一个 Hub。」

自研新仓库成立的条件（相对补现有）：

1. 现有 `web/` 无法改（无权限、或必须给外部客户白牌 SDK）。
2. 数据模型必须和 logcenter 切断。

两条补材料都未给出。默认路径：**在 kayou_tracker 上补 7 项**，仍是「自己的 SDK」，数据仍进 logcenter。

不成立：为了对标文章再写一个 Hub+七插件的 npm 包，而 tracker 已经分离采集与上报。

### 落地（生产清单 = 缺口 7 项）

按文章课纲、对照表「无/部分」：

1. 采样（error 全量，性能/API 可配）
2. 失败队列上限（删最早；文章口径 500）
3. 卸载发送改/加 `pagehide`（保留 visibilitychange；不要只靠 beforeunload）
4. 默认脱敏（password/token/authorization/cookie/secret）
5. CLS / INP + 白屏插件
6. eventId 幂等（UUID v7，重试复用；服务端 UNIQUE）
7. PerformancePlugin 等 Observer 的完整 `dispose`

v1 不含：AI 自愈、IndexedDB 替换 localStorage（可列为 v1.1）、fetch keepalive / Image 第四级（sendBeacon+XHR 已有，文章第四级是兜底）。

体积验收仍要补：gzip 与长任务占比，tracker 目前只有字段截断。

### 发散（仍独立于后台排期）

- 只补 1–4、7（可靠性），Vitals/白屏放到下一轮。
- Web 与小程序共用 Core，Flutter 继续自己的 jsonl 缓存。
- 镜像 Codeup → GitHub 私有仓，Cloud Agent 才能自己读 `Tracker.ts`，不再靠贴表。

## 连线

无其它存活 Brief。VibeHub 簇已删，无关。

## 产品信号

PC 浏览器监控的「自己的生产 SDK」= **kayou_tracker web/ + 7 项缺口 + logcenter**。市场空位不在再做一个通用 Hub；Sentry/ARMS 已在。差异化是卡游字段与数据不出域。Win32 不在本簇。
