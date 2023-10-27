---
title: "flunetd-server 無法寫入 log 到 open-search"
date: 2023-10-27
draft: false
description: "記錄在 open-search 上遇到的問題"
---

## 前言
前陣子值班時碰到 open-search 無法正常地寫入 log。以下是 RD 人員反應的截圖

可以看到下面這張圖，最新的 log 只記錄到 10/09 號。
![Log can't write](/img/opensearch/optimize/log.png)

## 排查
從源頭開始排查發現，fluentd-server 在噴 error log，如下面所示。

下面這段 error 的意思是 fluent 要傳入新的 log，open-search 需要再增加 24 個 shards，但已達到最大分片數 [4997/5000]
```
Validation Failed: 1: this action would add [24] total shards, but this cluster currently has [4997]/[5000] maximum shards open
```

{{< alert >}}
shards 的問題其實已經遇到很多次了，但是現在已經有使用 workflow 來管理 index 跟 shards 了，為什麼還會遇到分片不足的狀況呢？
{{< /alert >}}

shards 的堆積跟 index 肯定脫離不了關係，所以我們必須檢查 workflo他建立的 policy 是不是都有好好運作。

![policy for index](/img/opensearch/optimize/policy-for-indices.png)

接著就發現有大量的 `job status` 呈現 fail 的狀態，查看詳細的原因發現。大部分的 policy 會失敗都是因為 timeout。

所以推測可能是效能不足，導致 policy 處理起來不夠快速，間接的導致 shards 堆積。:backhand_index_pointing_down:(下圖已經調整過資源)

![叢集資源圖](/img/opensearch/optimize/cluster.png)

調整 cluster 資源，再重新 running 一次 policy 基本上就可以 success 了。也看到許多過期的 index 也 delete 了。

shards 減少了幾乎快一半，log 也成功地繼續寫入了！

![shards 下降](/img/opensearch/optimize/shards.png)

## 反思

之前有採用提高節點能容納的最大分片數，做暫時的解答。但假如一直讓叢集有大量分片的話其實也會影響效能，造成 RD 在使用時體驗不好，嚴重可能會整個 cluster 崩潰。