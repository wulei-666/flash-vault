# flash-vault

闪念正本。手机随时扔、云端展开、电脑（和手机）做决定。  
规则在 `AGENTS.md`，不要写进业务仓库。

## 怎么用

### 手机（Cursor Cloud Agent）

1. 选中本仓库，开一个 Cloud Agent（当天第一条闪念）。
2. 把只言片语或文章/视频链接直接发给它。
3. 看它回的 **Brief 头**，需要拍板就回一句：今天做 / 归档 / 升级为产品研判 / 并到某份。
4. 当天后面几条：在**同一条** Agent 里说「又一条闪念：……」，不要每条新开 Agent。

电脑可以关机。文件在 GitHub，不在这台电脑磁盘上。

### 电脑

```bash
git pull
```

打开 `queue.md` 看待决定，打开 `briefs/` 读全文、深追问。改完 push 回去。

### 工作画像

先改 `profile.yaml`。LearnLand 靠它写「明天能做」。独立产品 idea 不受当前项目过滤。

## 目录

| 路径 | 是什么 |
|---|---|
| `seeds/` | 每条闪念原文 |
| `briefs/` | 一簇一份活文档 |
| `cards/` | 连线用的判断句 |
| `sources/` | 抓取的正文 / 转写 |
| `queue.md` | 待决定队列 |
| `inbox/` | 备用：丢文件当闪念 |

## 第一次上 GitHub

本仓库需要是**私有**远程，Cloud Agent 才能克隆和 push。在本目录：

```bash
git init
git add .
git commit -m "初始化闪念仓库结构与 Agent 指令"
# 然后创建私有 repo 并 push（GitHub / GitLab 均可）
```

Cursor 账号需已连接该远程。手机仓库列表来自 cursor.com 上已接好的源码托管。
