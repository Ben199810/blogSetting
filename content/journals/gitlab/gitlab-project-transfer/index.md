---
title: "gitlab 專案搬遷"
date: 2023-11-27
draft: true
description: "在2023-10 ~ 11月之間，大型專案搬遷遇到的問題及從中學會的技能"
tags: ["gitlab"]
---

## 前言
因為公司建立一個統一管理 Repo Project 的 Gitlab，所以我們要將自己組內自建 Gitlab 上面的專案搬遷到統一管理的 Gitlab 上面。

搬遷方式：

1️⃣ Gitlab Mirror
Mirror 就是鏡像倉庫的意思，可以將目前開發人員推送程式碼的儲存庫，再推送到另一個遠端的`儲存庫`實現同步兩個儲存庫的 commit 進度及分支。