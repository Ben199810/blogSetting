---
title: "使用 CP 複製檔案資料，造成 Disk 被塞滿"
date: 2023-11-28
draft: false
description: "紀錄 2023/11/27 發生的線上問題"
tags: ["linux", "disk", "p2", "jenkins"]
---

## 前言
在 2023/11/27 這天，一個快下班的午後，發生的線上問題。所以特此紀錄 🥲

最近依舊在進行 GitLab 專案的搬遷，在 11/23 號時，嘗試轉移一個大約有 21GB 的專案，但過程中不是很順利。在 GitLab Import 的當下選擇了撤退。撤退的過程中由於專案的部署方式，不是使用 image。所以需要異動更改的部分也是異常的多。當天撤退其實有漏改 Jenkins 上的組態設定，導致 Job 執行 Fail。雖然沒有引發線上問題，但仍心有餘悸。

## 與 RD 討論
由於 Jenkins 部署的範圍是地端的 VM，也是現在大部分對外的服務流量的機群，對於 Jenkins 上的設定希望能不異動就不要異動。減少人為意外的發生。

❗️所以討論出一個結果❗️，先建立新的 Jenkins Job 這樣一來有問題，也可以直接切回舊 Job 立刻部署。即先建後拆的方式

但是有著很多的繁瑣的前置準備，這就是這場噩夢的開始。

我們的專案結構是透過 Jenkins 發佈機上的專案資料夾使用 Git 更新後再發佈給其他的 VM 大概會向下面的結構圖。

{{< mermaid >}}
flowchart LR

jenkins-push([jenkins發佈機])
project-file(專案資料夾)
gitlab(GitLab 儲存庫)

subgraph " "

jenkins-push -.- project-file
gitlab -.git pull.->project-file
jenkins-push --發佈--> VM1
jenkins-push --發佈--> VM2
jenkins-push --發佈--> VM3

end

{{< /mermaid >}}

## 導火線

每個 Jenkins Job 都會有自己的資料夾，假如要先建後拆的話，我就必須要複製 `prod-aliyun_captcha-newcasino` 資料夾內容到 `prod-aliyun-captcha-newcasino` 👉 注意這邊只是底線不一樣
![Jenkins Job 下會有自己的 file dir](/img/vm/jenkins-slave-file.png)

至於上述提到我需要先建立一個新的 new file 裡面是有著 21G 左右的專案，所以我直接用 CP 複製到新的 new file 不就可以了嗎？

🚨 接著下面的操作是血淋淋的教訓 🚨

我嘗試以下指令 `cp -af prod-ball_member-newcasino/* prod-ball-member-newcasino` 但此時我發現，隱藏的檔案沒有被 CP 例如: .git、.gitlab-ci.yml。因此我嘗試下列指令 `cp -af prod-ball_member-newcasino/{*,.*} prod-ball-member-newcasino` 👉 發現我用集合把前面是 `.開頭` 的隱藏檔案也想要 CP。

🛑 但是！好像有什麼東西混進來了！🛑 我們先用 `du` 來查看我們 CP 了哪些檔案。
![du 指令檢查檔案](/img/vm/du-check-file.png)

這樣竟然會複製上一層的資料，可以看到容量非常的可觀。隨之而來的就是 Disk 容量用滿的告警了。
![disk 使用狀況](/img/vm/disk-use.png)

## 炸彈
同一時間，RD 也有新的程式要發佈，這時候的 jenkins 就出現問題了。
![jenkins 錯誤訊息](/img/vm/jenkins-job.png)
雖然之後重新發布就沒有問題了，但還是一個教訓。

## 改善與檢討
這邊簡單回顧一下 `cp` 指令可以怎麼做。

如果要複製目錄的話，可以使用 `cp -r`，可以複製目錄跟子目錄底下的所有文件。
