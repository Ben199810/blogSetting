---
title: "container veth name provided (eth0) already exists 故障排除"
date: 2023-09-17
draft: false
description: "紀錄在工作時踩的坑，關於 CNI 底層"
tags: ["k8s", "pod", "node", "cni"]

---
## 前言
最近在協助公司專案轉雲端架構時，由於服務只能啟用一個 pod 提供線上服務的運作，因此也選擇使用 `statefulSet` 部署服務，在這過程中發現的問題。

## 事件流程
RD 同仁更新了 code 到 gitLab，並沒有順利的完成 CICD。原因是 statefulSet pod 在關閉時，停留在 `terminating`，雖然 k8s 有 `terminationGracePeriodSeconds`，但由於情況特殊。當下的 terminationGracePeriodSeconds 是 14400，長達四小時。

因線上緊急的問題，所以針對 terminating pod 採取了 `kubectl delete pod [pod name] --grace-period=0 --force`。之後重新建立的 pod 就會出現以下的錯誤訊息 :backhand_index_pointing_down:

{{< alert icon="fire" cardColor="#e63946" iconColor="#1d3557" textColor="#f1faee" >}}
container veth name provided (eth0) already exists
{{< /alert >}}
