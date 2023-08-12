---
title: "Ingress"
date: 2020-08-06
draft: false
description: "k8s ingress"
tags: ["k8s"]

---
## 介紹 ingress 
ingress 是 k8s 給 service 提供外部訪問的 URL、SSL、路由等功能。 可以理解為 nginx 或 traefik 等的代理工具。允許透過某個 URL 進入對應的 service，且碰到某些 route 能夠 proxy pass 到其他 service。

:star: ingress 又分成外部負載平衡、內部負載平衡。兩者在使用設定上有一些小小的不同。

## 外部負載平衡
通常對外部的負載平衡入口，都會設置防火牆或是白名單管理但是 k8s ingress 卻不能直接實現。需要透過一些 gcp 元件功能，來設置防火牆或白名單管理。

### cloud armor
armor 的翻譯是指盔甲的意思，這是由 gcp 提供的服務，具有分散式阻斷攻擊(DDos)防護機制、網路應用程式防火牆。可以搭配 loadBalance、Cloud CDN 來強化網路安全服務。

那麼我們要如何將 cloud armor 應用在我們 ingress 上面呢？接著看下去 :sunglasses:

首先我們可以先找官方的使用手冊 [Configuring Ingress using the default controller](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#configuring_ingress_features)

接著將重點放在 `backendConfig` 設定上面，前面的介紹已經知道 ingress 負責依據路由規則代理轉發。文件上 `backenConfig` 與 `service` 可以耦合。使用 backendConfig 的設定。

![BackendConfig and FrontendConfig overview](/img/k8s/ingress/backend-frontend-config.png "BackendConfig and FrontendConfig overview")

詳細想知道 backendConfig 設定範例可以在文件連接內找到 :point_right: [Configuring Ingress features through BackendConfig parameters](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#configuring_ingress_features_through_backendconfig_parameters)

接著下面繪製一張外部負載平衡的架構圖。可以看到下圖中 114.1.46.157 的客戶端在訪問的時候被 ingress 給擋下了。

原因是 ingress 後端 service 有耦合 `backendConfig`，透過 backendConfig 使用了 cloud armor。

{{< mermaid >}}
flowchart LR

cli1(client 34.41.26.173)
cli2(client 114.1.46.157)

cli1 --> ing
cli2 --x ing

subgraph k8s-cluster

ing((ingress))
svc-web(web-service)
svc-api(api-service)
df-backend(backend-config)
armor{{cloud-armor}}

ing --> svc-web
ing --> svc-api
armor -.- df-backend
df-backend -.-> svc-web
df-backend -.-> svc-api

end
{{< /mermaid >}}