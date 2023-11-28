---
title: "部署 GKE GitLab Runner"
date: 2023-11-22
draft: false
description: "記錄接觸如何建立 gke runner 的過程與問題"
tags: ["gke", "gitlab", "runner"]
---

## 前言
因為目前，組織內部在部署 gke 相關的服務時，runner 都還是採用 VM 執行 Pipeline Job。如果因為大量的 Job 導致 runner 的速度變慢的話，就需要擴展 VM 數量。這樣一來總體花費會變高。

經過一次架構上面的調整，GitLab Runner 設計成了主從式的架構，由 `Runner Manager` 去啟動 Runner VM 執行 Pipeline Job，結束時，Runner VM 會關機。這樣做的好處是
常駐只需要一台 Manager VM，相較之前省錢。

如下圖 👇 可以看到下面有`執行中的 VM`、`啟動中的 VM`、`關閉中的 VM`
![vm manager 示意圖](/img/gitlab/gke-runner/vm-manager.png)

或許我們可以有更好的做法？沒有錯! [Gitlab 官方的文件](https://docs.gitlab.com/runner/install/kubernetes.html?source=post_page-----6abfb110d9da--------------------------------#configuring-gitlab-runner-using-the-helm-chart)，有說明使用 Helm 部署 Gitlab Runner 到 K8S ❤️

`values.yaml` 預設值的設定可以在 [Chart 儲存庫](https://gitlab.com/gitlab-org/charts/gitlab-runner/blob/main/values.yaml) 中找到。

## 設定與配置
為了使 runner 正確的運行，values.yaml 需要注意幾項配置。

1️⃣ `gitlabUrl`: 用於註冊運行程式的 GitLab 伺服器完整 URL。例如，https://gitlab.example.com。

2️⃣ `rbac`: 為 GitLab Runner 建立 RBAC 規則，以建立要在其中執行作業的 pod

* ⭐️ 如果您的叢集啟用了 RBAC，您可以選擇讓圖表建立自己的服務帳戶👇
```yaml
rbac:
  create: true
```

* ⭐️ 若要使用現有的服務帳戶，請使用👇
```yaml
rbac:
  create: false
  serviceAccountName: your-service-account
```

3️⃣ `runnerRegistrationToken` 👉 (GitLab 15.6 中已棄用) 之後採用 `runnerToken`
![helm runnerToken 設定](/img/gitlab/gke-runner/runnerRegistrationToken.png)

## 部署
依照官方建議步驟執行 Script

```shell
# add repo
helm repo add gitlab https://charts.gitlab.io

# install
# CONFIG_VALUES_FILE -> 更換 values.yaml
helm install --namespace [NAMESPACE] gitlab-runner -f [CONFIG_VALUES_FILE] gitlab/gitlab-runner
```

完成後，從 kubectl get pod | grep gitlab-runner 就能看到 gitlab-runner 已經部署成功了！

而當你回到 Gitlab > Settings > CI/CD ，你也會看到 Specific Runner 掛載了一個 Runner 了。