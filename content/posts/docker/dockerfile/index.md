---
title: "Dockerfile"
date: 2023-08-27
draft: false
description: "dockerfile 相關知識"
tags: ["docker"]

---
## 前言
docker 是將 image 俗稱映像或者鏡像檔，建立容器的模擬使用環境的技術。那要如何建立 image 呢？

接著介紹本篇文章的重點 Dockerfile

## Dockerfile
Dockfile 是一個建構 image 的腳本，基於你選擇的基底映像，開始安裝在開發環境所需要的依賴以及套件等等...

建立 image 也可以保證每個開發者在進行開發時使用的環境都相同，不會導致將程式碼推上去版控，經過打包發布才發現有相容性的問題。有效的消除了在協同開發時會遇到的衝突呢～

下面提供簡單的 dockerfile 範例。下面的範例步驟使用引用 alpine image 當做基底，並且安裝 curl 套件。以 image 啟動 container 時，會執行 /bin/bash。 :point_right: `docker run -it [image] /bin/bash`

```dockerfile
# image 基底
FROM alpine
# 安裝套件或執行的 shell
RUN apk add curl

CMD ["/bin/bash"]
```

## 指令

### From
在 dockerfile 中，建立多映像。

```dockerfile
FROM image
FROM image:tag
```

### Maintainer
說明維護者訊息

```dockerfile
MAINTAINER <name>
```

### Run
前者將在 shell 終端中運行命令，即 /bin/sh -c；後者則使用 exec 執行。

指定使用其它終端可以透過第二種方式實作，例如：RUN ["/bin/bash", "-c", "echo hello"]。

每條 RUN 指令將在當前映像檔基底上執行指定命令，並產生新的映像檔。當命令較長時可以使用 \ 來換行。

```dockerfile
RUN command
RUN ["executable", "param1", "param2"]
```

### CMD
CMD 用來在 container 啟動時，執行的指令。一個 Dockerfile 只能有一個 `CMD` 這很重要。如果一個 Dockerfile 有多個 CMD，只有最後一行 CMD 會被執行。

⭐️ 通常也會當作 ENTRYPOINT 指令執行時的參數。

如果使用者在使用 image 時，有指定啟動指令，也會覆蓋 `CMD`

```dockerfile
CMD [ "/bin/sh" ]
```

### ENTRYPOINT
這邊要注意 `ENTRYPOINT` 跟 `CMD` 很像，但是使用情境不一樣。

{{< alert cardColor="#e6c229" iconColor="#000000" textColor="#000000" >}}
ENTRYPOINT 用於 container 啟動之後執行的指令，並不會被 docker run 給覆蓋！
{{< /alert >}}

Dockerfile 中也只能有一個 `ENTRYPOINT`，如果一份 dockerfile 中有多個，只有最後一個會生效。

💼 [官方文件](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact) 有提供表格來對比 CMD 跟 ENTRYPOINT 之間的交互關係。

### EXPOSE
docker container 對外開放的埠號，提供外界使用。

```dockerfile
FROM PHP
EXPOSE 9000
```

## 參考文章
[Docker - 從入門到實踐](https://philipzheng.gitbook.io/docker_practice/dockerfile/instructions)