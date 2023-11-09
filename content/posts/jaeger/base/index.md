---
title: "jaeger 介紹"
date: 2023-11-07
draft: false
description: 
---

## 前言
jaeger 是一個開源的分散式跟蹤 Trace 系統，可以建構雲服務生態中，可觀測性服務重要的工具之一。

## jaeger 架構
低 TPS 的需求下，我們可以使用以下架構來快速地達成部署。
![](/img/jaeger/jaeger-system1.png)

高 TPS 的需求下，我們可以使用以下架構來快速地達成壓力緩衝增加系統可用性。
![](/img/jaeger/jaeger-system2.png)

以上兩張架構圖可以了解到幾個 jaeger 重要的組件，下面一一介紹：

:one: Client

:two: Agent

:three: Collector

:four: Storage :star: 默認是使用 Memory

:five: Query service and UI :backhand_index_pointing_right: 從storage中檢索trace data，並提供React搭建的UI與存儲的資料做交互。

## Jaeger 的基本邏輯單元
想要了解基本的邏輯單元，我們要先理解 Trace 跟 Span。

:one: Trace 就是系統執行的路徑

:two: Span 是最小的單位，資訊包含起始時間、執行時間，span 有可能是有序的或者是巢狀的

![](/img/jaeger/jaeger-trace.png)
