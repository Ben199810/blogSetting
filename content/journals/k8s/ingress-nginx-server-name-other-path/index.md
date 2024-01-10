---
title: "ingress-nginx 無指定網址路徑導轉設定"
date: 2024-01-10
draft: false
description: "ingress-nginx 如果有無指定的網址，要如何設定路徑導轉規則呢？"
tags: ["k8s", "ingress-nginx", "ingress"]
---

## 前言
近期公司入口的架構，正在由 `nginx-proxy` 轉向由第三方開源支持的 `ingress-nginx` 還是有遇到一些不熟悉的問題，寫起來紀錄一下。

今天與遇到一個需求，公司 SR 負責建立的 CDN 要連 `ingress-nginx` 入口。但因為 CDN 網址有不固定的特性。所以 ingress-nginx 預設如果認不出 CDN 網址就會導轉 `nginx 404` 的頁面。導致沒辦法連到 ingress-nginx 後端服務讀取圖檔。

{{< mermaid >}}
flowchart

subgraph " "
  domain(original domain)
  cdn-domain(xxx.xxx.com)
  ingress-nginx((ingress-nginx))
  backend-service(service)
  not-found(404 page)

  domain --> ingress-nginx --> backend-service
  cdn-domain -.-> ingress-nginx -.無定義host.-> not-found
end

{{< /mermaid >}}

## 解決過程
這個過程我真的是繞一大圈，但當作經驗記錄下來吧。畢竟也是有學到東西XD

### ingress host 設定
一開始，設定都有指定類似 `*.test.net` 這類的網址設定。因為可以使用正規表示法，所以一開始很異想天開寫了以下設定。

```yaml
- host: "*"
```
結果就是噴了以下錯誤，就是提示 host 的格式驗證不對，是不能這樣填寫的。

{{< alert icon="fire" iconColor="#000000" cardColor="#e63946"  >}}
Error: UPGRADE FAILED: cannot patch "ingress" with kind Ingress: Ingress.extensions "ingress" is invalid: spec.rules[4].host: Invalid value: "*": a wildcard DNS-1123 subdomain must start with '*.', followed by a valid DNS subdomain
{{< /alert >}}

### helm default backend
helm values.yaml 中有 `defaultBackend` 可以設定，當 `enable` 參數是 true 時，部署時會多建立一組 `service + deployment`。

defaultBackend 更多的詳細設定: [values.yaml](https://github.com/kubernetes/ingress-nginx/blob/05d68a15127b437891c0250f0d7cf7b5b9f84b92/charts/ingress-nginx/values.yaml#L916C6-L916C6)

但看[文件](https://kubernetes.github.io/ingress-nginx/user-guide/default-backend/) 設定 defaultBackend 更像是製作 custom error page。

而且預設的 paths 也被配置好了，沒有可以更動的空間。
- `/healthz` that returns 200
- `/` that returns 404

```yaml
defaultBackend:
  enabled: false
```

### custom nginx template
[文件](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/custom-template/#custom-nginx-template)有提到可以客製 `nginx.tmpl`

如果要製作 nginx.tmpl 就必須要整份文件下去改。因為 ingress-nginx 是透過這個 tmpl 檔案來生成 `nginx.conf`

官方的 values 也有提個對應的方法讓我們可以用 configmap 的方式掛載

```yaml
{{- if .Values.controller.customTemplate.configMapName }}
  - name: nginx-template-volume
    configMap:
      name: {{ .Values.controller.customTemplate.configMapName }}
      items:
      - key: {{ .Values.controller.customTemplate.configMapKey }}
        path: nginx.tmpl
{{- end }}
```

## 結論
後續在 k8s 官方的[文件](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/#ingress-rules)發現 `host(可選)` 👈 這的詞真的關鍵。

於是建立的一個 test-ingress.yaml 來測試 ingress-nginx 是否會吃到異動。

最後測試結果: 沒有問題 ✅ 下圖設定已經吃到異動的設定了 👇

![設定異動圖](/img/k8s/ingress-nginx/server-name-default-setting.png)