---
title: "helm 使用 gcp artifact registry 存放 chart"
date: 2023-11-10
draft: true
description: "helm 打包好的 chart 如何配合 gcp artifact registry 存放"
tags: ["helm", "gcp"]
---

## 前言
之前實作 helm 的 chart 存放區時，使用 gitlab 的 `Package Registry` 但後續遇到不同 subgroup 的專案需要存取需要額外設置公有金鑰，十分的不方便。後來在其他同事的 demo 看到把存放區設置為 `GCP Artifact Registry`

## 對 Artifact Registry 進行身份驗證
[官方參考文件](https://cloud.google.com/artifact-registry/docs/helm/store-helm-charts#auth)

:one: 預設情況下，Helm 支援 Docker 設定檔 config.json中的註冊表設定。

:two: 使用存取權杖進行身份驗證

```shell
gcloud auth print-access-token | helm registry login -u oauth2accesstoken --password-stdin https://us-central1-docker.pkg.dev
```
* oauth2accesstoken是使用存取令牌進行身份驗證時使用的使用者名稱。
* gcloud auth print-access-token是取得存取權杖的 gcloud 命令。您的存取令牌是用於身份驗證的密碼。

## 推送圖表
```shell
helm push hello-chart-0.1.0.tgz oci://us-central1-docker.pkg.dev/PROJECT/quickstart-helm-repo
```

## 使用圖表部署
```shell
helm install hello-chart oci://us-central1-docker.pkg.dev/PROJECT/quickstart-helm-repo/hello-chart --version 0.1.0
```

## 總結
我是覺得 google 提供的文件很完整，而且也不難所以採用這方式。但其實 gitlab 也是差不多要設置一組 Job Key 這樣才能使 subGroup 下的專案正常使用 Helm Chart。