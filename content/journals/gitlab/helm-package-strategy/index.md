---
title: "優化打包 helm chart 分支策略"
date: 2023-11-20
draft: false
description: "優化 ci 在 package 的方式以及增進使用效率"
tags: ["linux", "gitlab", "ci", "helm"]
---

## 前言
之前有日誌記錄到，我們是如何打包 Helm Chart 到 GCP Artifact Registry，但是因為對於一個 Chart 的管理，公司組內是採用三個版本去更新，也因為這樣當初調整 CI 的時候，直接用很陽春的方式，切出了三個一樣的檔案去管理。

如下圖所示 👇

![](/img/gitlab/helm/chart-file-before.png)

管理幾次之後就發現，如果要推送三個環境，就要更改檔案三次、推送三次。

{{< alert cardColor="#e6c229" iconColor="#000000" textColor="#000000" >}}
非常的煩瑣跟如果有缺漏的話容易造成，三個環境從開發到正式上線的不一致！
{{< /alert >}}

## 優化
為了解決上面提到的問題，我需要優化 CI 觸發的條件，並且希望只要管理一份檔案，並用合併的方式去更新個環境的 Template 來達到一致性！

新的 CI 流程透過分支來觸發，並且會透過對應的分支打包成不同版本的 Chart。這樣也只需要管理一份檔案。不必再分成三份！

如下圖所示 👇

![](/img/gitlab/helm/strategy.png)

## 過程中遇到的問題
在 Helm 的檔案結構中，Chart.yaml 用來管理版本資訊跟相依的子 Chart。如果需要打包成不同版的 Chart 那這樣的話，我們需要透過給予變數的方式做些變化，於是我在 CI 上做了一些手腳。

⭐️ 這邊會用到一個指令 `envsubst` 👉 要預先安裝 `apt-get gettext-base` ， `gettext-base` 我已經在 dockerfile 安裝了，所以 CI 不用安裝。

1️⃣ 首先我們在 Chart.yaml.tpl 挖空。🚨 這邊我是用 `.tpl` 結尾代表這是模板。

![](/img/gitlab/helm/envsubt-values.png)

2️⃣ 接著，在 helm package 步驟前，先建立我們需要的 Chart.yaml

⭐️ 下面的語句是指，將變數填入 Chart.yaml.tpl 檔案並匯出 Chart.yaml～

![](/img/gitlab/helm/envsubst-use.png)

這樣就能解決，需要管理三份檔案的問題與麻煩了！