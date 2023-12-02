---
title: Docker - Docker Tags (標籤)不只是標籤
tags:
  - Docker
  - 部署
  - tag
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-22 01:24:12
---

在 Docker Container(容器) 篇，我們曾短暫的介紹過 Docker Tags ，但那只是最表面的，但其實 Docker Tags(標籤) 它不只是標籤，今天我們就來深入了解一下 Docker Tags(標籤) 還有哪些功用吧？

先來複習一下之前說到的當我們想要啟動一個 Container(容器) 時，打了 `docker run ruby` 時，因為本地沒有叫做 ruby 的 Image(映像) ，所以他自動去遠端儲存庫找 `ruby:latest` 這個 ruby Image(映像)
，以及使用預設的 latest Tags(標籤) ，所以我們了解到我們可以在 Docker Hub 看到及選擇 pull 或 run 不同 Tags(標籤)的 ruby Image(映像)，但這些其實都是在使用別人建好的 Image(映像) 或是 Tags(標籤) ，其實我們也可以建立屬於自己的 Tags(標籤) 呦！

## 建立自己的 Tags(標籤)

首先我們可以先使用

```docker
docker images
```

看一下自己現在本機所有的 Image(映像)

![看本機所有的 Images](/image/dockerDay7/7_1.png)

這邊我們可以看到有之前我們 run 時，從公共儲存庫 pull 下來的 ruby Image(映像) ，這時我們可以使用以下語法來建立一個有新標籤的 Image(映像)

```docker
docker tag <本機現有映像 id_or_repository>:<本機現有標籤> <新映像 repository>:<新標籤>
```

![建立一個有新標籤的 Image](/image/dockerDay7/7_2.png)

這個語法是為本機原本的 ruby Image(映像)，建立一個新的 Image(映像) ，而這個 Image(映像)是新的標籤 `v1`，所以當你看 `docker images` 時，會看到兩個 ruby Repository(倉庫)，但是是不同 Tags(標籤) - `latest` 跟 `v1`。

另外建立 Tags(標籤) 成功並不會有輸出。

📍 注意事項：
Tags(標籤)可以包括字母、數字、橫線(`-`)和下劃線(`_`)，但不建議使用特殊符號(`.`、`:`、`/`等)。

新的映像 repository 跟新標籤名稱當然可以取自己滿意的名字，如圖

![repository 跟新標籤名稱可自訂](/image/dockerDay7/7_3.png)

這時我們有兩個不同 Tags(標籤) 的 ruby Image(映像)，若是我們想將 Tags(標籤) 為 latest 的 ruby Image(映像) 建立一個新的 myapp Repository(倉庫) 且 Tags(標籤) 為 latest 時，因為是默認的 latest Tags(標籤) ，所以可以使用簡寫

```docker
docker tag <本機現有映像id_or_repository> <新映像 repository>
```

![使用簡寫建立新的 Repository](/image/dockerDay7/7_4.png)

## 建立＋推送(push)自己的 Tags(標籤)

```docker
docker tag <本機現有映像id_or_repository> <Docker 儲存庫的主機名或 IP 地址>:<端口>/<新映像 repository>:<新標籤>
```

這個指令包含兩個動作

1. 就是我們上面介紹的建立新的 Tags(標籤)
2. 還包含了將這個新 Tags(標籤)的 Image(映像)，推送到指定的 Docker 儲存庫 (myregistryhost:5000) 中，以便在遠端的主機上使用或與其他人共享。

那這部分我們之後也會詳記介紹到，今天 Docker Tags(標籤) 就介紹到這邊啦～我們明天繼續！
