# kayou_tracker 对照补材料

来源：用户 2026-08-17 贴回本会话。Cloud Agent 未克隆 Codeup 仓库；下表由用户本机读源码填写。

远程：`git@codeup.aliyun.com:kayou/qianduan/kayou_tracker.git`  
本机路径：`/Users/arvin/ky/kayou_tracker`（Cloud Agent 读不到，属设计限制）

依据文件（用户标明）：`web/src/core/Tracker.ts`、`ErrorPlugin.ts`、`PerformancePlugin.ts`、`web/README.md` TODO。

---

**Cloud Agent 读不到 `/Users/arvin/ky/kayou_tracker` 是设计如此，不是这次会话配错。** 它只从已连接的 GitHub / GitLab / Bitbucket / Azure DevOps **克隆远程仓库**，看不到你这台 Mac 的磁盘。[官方](https://cursor.com/docs/cloud-agent.md)

本机上这个仓库的 `origin` 是：

`git@codeup.aliyun.com:kayou/qianduan/kayou_tracker.git`

所以它去 `gh repo list wulei-666` 也找不到。Codeup 不在 Cursor Cloud Agent 的克隆来源里。

---

### 先定边界（源码已经能定）

`kayou_tracker` **不是** Windows/C++ 原生 PC SDK。根 README 写明三套：

| SDK | 目录 | 宿主 |
|---|---|---|
| Web | `web/` | 浏览器 / Vue |
| 微信小程序 | `web/src-mini/` | 小程序 |
| Flutter | `app/` | Dart，经 MethodChannel 收 Native 异常 |

和微信文是同一条链路：**Web/H5/小程序监控 SDK**。你那句「生产级 PC SDK」若指 Win32 dll，对照物选错了；若指给 PC 浏览器里的中后台用，对照物就是 `web/`。

---

### 对照表（本机读源码填的，不是空口）

依据：`web/src/core/Tracker.ts`、`ErrorPlugin.ts`、`PerformancePlugin.ts`、`web/README.md` TODO。

| 检查项 | 文章标准 | kayou_tracker（Web） |
|---|---|---|
| 采集与上报分离 | Hub + Plugin | **有。** Tracker 管队列/上报；Performance / Behavior / Api / Error / Track 插件采集 |
| 插件 teardown | teardown 防重复监听 | **部分。** `Tracker.destroy()` 调插件 `dispose`；ErrorPlugin 有 `removeEventListener`。README TODO 仍写「销毁？」；PerformancePlugin **未见** dispose Observer |
| 早于业务的调用 | init 前 API 入队 | **无独立预队列。** `new Tracker()` 即注册插件并 `pushLog`。`track()` 在未开 Track 插件时直接 return |
| 卸载仍能送出 | pagehide + sendBeacon | **部分。** `sendLogs` 优先 `sendBeacon`，失败再 XHR。发送绑在 `beforeunload` 和 `visibilitychange=hidden`，**不是**文章强调的 pagehide。无 `fetch keepalive`、无 Image 第四级 |
| 本地持久化 | IndexedDB，上限 500 | **localStorage** 键 `trackerFailedLogs`。失败重试 3 次再写入。代码 TODO：超出最大数量删最早的——**未做上限** |
| 采样 | error 全量、性能 0.3 | **无。** README TODO：「采样率？」 |
| 幂等 | UUID v7 | **无 eventId。** `sessionId` / `visitId` 为时间戳+随机 |
| 隐私 | beforeSend，password 等过滤 | **部分。** `filter` 拦截、`use` 改字段（类似 beforeSend）。README TODO：「日志压缩 / 脱敏 / 加密」——**未做**默认密钥过滤 |
| 体积/开销 | gzip <15KB / 长任务 ≤2% | 有 `maxFieldLength`、超大 body 截断。**未见**包体/长任务占比验收 |
| 性能指标 | LCP/INP/CLS 等 Web Vitals | PerformanceObserver 收 **LCP/FCP/FP**、长任务、资源。**未见 INP/CLS** |
| 白屏 | 根节点高度/子节点 | **无** WhiteScreen 插件 |
| SPA PV | 劫持 pushState | BehaviorPlugin 有路由相关监听（含 `pagehide`）。Vue 后台需对照是否劫持 history |
| 数据去向 | 文中自有上报 | 默认打到卡游 `logcenter`（自有数仓，不是 Sentry 出域） |

Flutter 侧另有文件缓存（`path_provider`、jsonl、约 2MB 轮转）、冷启动/回前台重传、Native crash 要自己往 `tracker/native_error` 扔。那是 App，不是 Win32 SDK。

---

### 和文章比，缺的生产件（Web 路径）

文章当课纲、tracker 当现有实现，缺口是：

1. 采样  
2. 失败队列上限  
3. 卸载用 pagehide 而不是只靠 beforeunload  
4. 默认脱敏  
5. CLS/INP、白屏  
6. eventId 幂等  
7. 插件 Observer 完整 teardown  

「再搞一套自己的生产级 SDK」在 Web 上更像是 **把 kayou_tracker 补到生产清单**，不是从零再写一个 Hub。原生 Windows 是另一摊，tracker 里没有。
