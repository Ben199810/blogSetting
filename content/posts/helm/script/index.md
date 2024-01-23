---
title: "helm - 強而有力的管理 k8s "
date: 2023-11-10
draft: false
description: "介紹 helm and 常用實用的指令"
tags: ["k8s", "helm"]
---

## 介紹
Helm 是一個用於簡化 Kubernetes 應用程式部署和管理的開源工具。它允許您定義、安裝和升級 Kubernetes 應用程式，同時處理應用程式所需的複雜性，如相依性、配置和版本控制。使得開發者和運維團隊更容易分享和重複使用應用程式配置。通過 Helm，您可以更輕鬆地構建、發佈和管理容器化應用程式，加速了在 Kubernetes 上的開發和交付流程。

Helm 的核心概念包括：

:one: **Chart**：Helm 包的稱呼。它是一個預定義的 Kubernetes 應用程式模板，包括配置、相依性和與應用程式相關的所有資源

:two: **Release**：安裝的 Helm Chart 的一個特定實例。每個 Release 具有唯一的名稱，您可以輕鬆地安裝、升級或卸載它。

:three: **Repository**：存儲 Helm Charts 的位置。您可以從公共或私有倉庫中獲取 Helm Charts，並透過配置 Helm 來指定使用哪個倉庫。

## 檔案框架
{{< mermaid >}}
flowchart

subgraph " "

CustomChart --- Chart.yaml
CustomChart --- Value.yaml
CustomChart --- Templates
Templates --- templates.tpl
Templates --- templates.yaml

end

{{< /mermaid >}}

### Templates
Templates 下面有兩種檔案，一種是 Yaml 一種是 Tpl

:one: 生成 yaml 輸出的模板文件在使用擴展名 .yaml

:two: 擴展名 .tpl 可以用於生成自訂的模板文件 :point_right: 很實用 ⭐️ ⭐️ ⭐️

```t
{{- define "nginx.fullname" }}
{{/* ... */}}
{{-  end }}
```

### Value
value 顧名思義就是"值"的意思，也就是用來存放"變數"的 yaml

value 可以是嵌套的，也可以是扁平的

:one: 嵌套的
```yaml
server:
  name: nginx
  port: 80
```

:two: 扁平的
```yaml
serverName: nginx
serverPort: 80
```

⭐️ 三種潛在的 value 來源:

* chart 的 value.yaml
* helm intall/upgrade -f 提供的 values 文件
* helm intall/upgrade 使用 --set 或 --set-string 傳遞的參數

## 常用指令

### 一般自訂 Chart 安裝
⭐️ **debug**：輸出安裝過程中的詳細資訊 ⭐️ **dry-run**：預覽安裝過程 ⭐️ **atomic**：更新過程中有任何一步驟失敗，回滾到先前狀態
```shell
helm upgrade --intstall --debug --dry-run [RELEASE_NAME] [CHART_PATH] --atomic
```

### 本機 repo 安裝
⭐️ **version**：指定安裝版本
```shell
helm install [RELEASE_NAME] [REPO_NAME]/[CHART_NAME] --version [VERSION]
```

### 印出部署預覽的 Yaml
🚨 有時候使用 --dry-run 的時候，因為已經有 release 管理`相同名稱`的資源了。這時候就可以使用 template 預覽，這樣就不會噴錯。
```shell
helm template --upgrade [RELEASE_NAME] [REPO_NAME]/[CHART_NAME]
```

### 列出已安裝 Release
```shell
helm list
```

### 新增 Chart Repo
```shell
helm repo add [REPO_NAME] [URL]
```

### 列出 Chart Repo List
```shell
helm repo list
```

### 更新 Repo
```shell
helm repo update
```

### 查看 Respository 有哪些 Helm Chart 可用
```shell
helm search repo [search_keyword]
```

### 查看 Chart 詳細版本
```shell
helm search repo [REPO_NAME] --versions
```

### 查看 releases 歷史版本
```shell
helm history [RELEASE_NAME]
```

### 退版到指定版本
```shell
helm rollback [RELEASE_NAME] [REVISION_NO]
```

### 打包 helm chart 
```shell
helm package [HELM_CHART_FOLDER]
```

### push 到 HTTP Repo
```shell
helm push [HELM_CHART_NAME]-0.1.0.tgz [REPO_NAME]
```

### 測試 & 驗證 Helm Chart
```shell
# 檢查 Helm Chart 是否有語法上的錯誤
$ helm lint [CHART_FOLDER]
```