---
title: "跨區域內部負載網路時遇到無法連線"
date: 2023-11-08
draft: true
description: "紀錄 SR 提供的 annotions"
---

## 前言
近期在工作上跟 SR 對接工作，需要透過專線網路連線至 GCP 的內部負載平衡，由於 GCP 專案一開始切分的網域比較特別。是 `新加坡` 的網段，導致一開始無法透過 `台灣` 的專線連線。

## 解決方式-手動
在 GCP 上面，使用 LoadBalance 的 SVC 會被分類在負載平衡。我們可以先找 SVC，進入頁面後，負載均衡的編號。

![](/img/k8s/journals/svc-id.png)

上面有一個編輯鍵

![](/img/k8s/journals/svc-id-1.png)

在前端設定的部分可以開啟 `全域存取權`

![](/img/k8s/journals/svc-id-2.png)

## k8s Setting
