---
title: "docker 疑難雜症系列"
date: 2024-01-13
draft: false
description: "在學習 docker 時碰到的問題在這裡記錄下來"
tags: ["docker", "question"]
---

## 為什麼 docker container 啟動後又立刻關閉退出？
剛開始學習 docker 因為我是直接使用 nginx、php 這類有持續進程(process)的 image。在實作時沒有遇到這類的問題。後來自己使用 alpine 為基底開始想要製作 docker image 時，發現我的 docker container 在啟動時，又立刻退出了。

⭐️ 首先我們要清楚的知道 docker 的原理。docker container 在啟動時，會執行的進程，稱為 process。如果 container 啟動時就執行完所有 process 的話，就會關閉接著退出。有關容器的啟動後需要執行的 process 牽涉到 dockerfile 的 `CMD` 跟 `ENTRYPOINT`，有興趣了解可以看以下文章複習。

{{< article link="/posts/docker/dockerfile/" >}}

**那為什麼有些 container 不會呢？**

### 啟動一個持續運行的程序
舉一個簡單的範例，像是 postgres image，我們可以使用 `docker inspect postgres` 來查看 image 的詳細資訊。可以看到容器在啟動後會預設執行 `postgres` 程序，這個程序會持續運作。

```
...
"Cmd": [
    "postgres"
],
...
```

### 啟動一個互動式 shell
有些鏡像的預設執行指令是啟動一個互動式 shell，直接執行 docker run 容器啟動後就會結束。以 `node` 鏡像來舉個例子，同樣先看一下鏡像的詳細資料。可以看到容器在啟動後會預設執行 node 程序，該程式會進入 node 的互動式 shell。

那麼對於這種情況，我們要怎麼讓容器啟動後不會停止呢？答案是使用`-i`、`-t` 或 `-it` 參數。

```
...
"Cmd": [
    "node"
],
...
```
