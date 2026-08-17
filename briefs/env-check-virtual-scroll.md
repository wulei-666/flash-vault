---
id: brief_env-check-virtual-scroll
cluster_id: env-check-virtual-scroll
title: Vue 长列表虚拟滚动（环境自检 Demo）
mission: LearnLand
status: ready
seed_ids:
  - seed_20260817_001
card_id: cards/env-check-virtual-scroll.md
related: []
source_gaps: []
updated_at: 2026-08-17T13:40:00+08:00
---

> 这是一份「环境自检 Demo」Brief：用来证明 flash-vault 的展开工作流在 Cloud Agent 里能端到端跑通（捕获 Seed → 判簇 → 路由 → 写 Brief/Card → 更新队列 → commit/push）。确认环境无误后可在合并前整簇删除。

## 1. 这是什么

一条 LearnLand 闪念：Vue 长列表（订单列表几千条）滚动卡顿，想学虚拟滚动（virtual scrolling / 窗口化渲染）怎么做，并评估能否用到电商中后台。

## 2. 主任务

LearnLand。信号词：「想学」「怎么用」「用到电商后台」——学会一个技术点并落到当前工作。

## 3. 和你的关系

对照 `profile.yaml`：角色前端工程师、栈 Vue + 电商中后台、在做 `kayou.ecshopx.admin`。订单/商品列表长表格是电商后台高频场景，虚拟滚动是直接可落地的性能手段。

## 4. 待你决定（最多 3 条）

1. 今天做：在订单列表接一个虚拟滚动库做个 demo 分支？
2. 归档：只是随手记，先不展开？
3. 升级：把「大数据量表格渲染方案」作为团队通用组件沉淀？

## 5. 探索地图

- 纵向：可视区窗口化渲染原理 → 固定高度 vs 动态高度 → 现成库选型 → 接入订单列表。
- 横向：分页 / 懒加载 / 后端游标 与虚拟滚动的取舍；与表格组件（如 el-table）的兼容性。

## 6. LearnLand 展开

- **是什么**：只渲染视口内可见的少量行（外加缓冲区），滚动时回收复用 DOM，避免一次性渲染几千个节点导致卡顿。
- **怎么用（Vue 生态）**：`vue-virtual-scroller`、`vue3-virtual-scroll-list` 等库提供窗口化列表容器；固定行高最简单，动态行高需测量。
- **明天能做**：在 `kayou.ecshopx.admin` 订单列表页开分支，用一个虚拟滚动库替换全量渲染，几千条数据对比滚动帧率。
- **排障视角**：先确认卡顿来自 DOM 节点数还是每行重渲染；若是后者，配合 `v-once` / 拆分单元格组件。

## 7. 连线

暂无其他簇可连线（首簇）。

## 8. 产品信号（短）

「电商后台大数据量表格性能方案」若沉淀成通用组件，可复用到多个中后台项目；是否值得独立产品化需另起 Compete 研判，不在本 Demo 范围。
