---
title: ingress-nginx 常用設定
date: 2024-01-05
draft: false
description: "ingress-nginx 常使用到的設定"
tags: ["k8s", "helm", "ingress-nginx"]
---
## SSL 重定向
SSL 重定向將對流量從 Http 重定向到 Https 非常有用。如果 ingress 有定義 TLS 證書。ingress-nginx 會重新導向(301)到 Https。可以透過以下設定停用。
```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "false"
```

如果沒有可用的 TLS 證書，也是可以強制重定向到 Https。
```yaml
nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

## 超時設定
nginx 中有許多 timeout 的設定，在 ingress nginx 中也可以設定這邊列出常用的幾個 timeout。

- proxy-sent-timeout
- proxy-read-timeout
- proxy-connect-timeout
- client-body-timeout
- client-header-timeout