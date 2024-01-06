---
title: "跨區域內部負載網路時遇到無法連線"
date: 2023-11-08
draft: false
description: "紀錄 SR 提供的 annotions"
---

## 前言
近期在工作上跟 SR 對接工作，需要透過專線網路連線至 GCP 的內部負載平衡，由於 GCP 專案一開始切分的網域比較特別。是 `新加坡` 的網段，導致一開始無法透過 `台灣` 的專線連線。

## 解決方式-手動
在 GCP 上面，使用 LoadBalance 的 SVC 會被分類在負載平衡。我們可以先找 SVC，進入頁面後，負載均衡的編號。

![](/img/k8s/journals/loadbalance-id.png)

上面有一個編輯鍵

![](/img/k8s/journals/loadbalance-edit.png)

在前端設定的部分可以開啟 `全域存取權`

![](/img/k8s/journals/loadbalance-global-access.png)

## k8s Setting

那一定要手動去設定嗎？還有 SVC 的設定要是變動了，`全域存取權 設定會不會失效呢？

:warning: 經過測試發現，如果更動 SVC 的 loadBalance 的白名單設定的話，如果你只有用手動去設定，那 `全域存取權` 就會失效，因此我們必須把這項設定寫在 k8s Yaml 上面才是最有效的

```yaml
apiVersion: v1
kind: Service
metadata:
  name: agent-gateway-ingress-nginx-controller-internal
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: agent-gateway
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.9.1
    helm.sh/chart: ingress-nginx-4.8.1
  annotations:
    meta.helm.sh/release-name: agent-gateway
    meta.helm.sh/release-namespace: ingress-nginx
    networking.gke.io/load-balancer-type: Internal
    # 全域內部負載平衡存取設定
    networking.gke.io/internal-load-balancer-allow-global-access: "true"
spec:
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
      nodePort: 31669
    - name: https
      protocol: TCP
      port: 443
      targetPort: https
      nodePort: 30961
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: agent-gateway
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
  loadBalancerIP: 10.1.5.16
  loadBalancerSourceRanges:
    - 10.0.0.0/8
```
