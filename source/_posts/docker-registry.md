---
title: Docker - Docker Registry (註冊表) 是倉庫的倉庫
tags:
  - Docker
  - 部署
  - registry
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-23 21:16:30
---

在前面介紹 docker 三寶時，有介紹到 Docker Repository（儲存庫），可以把它想像成存放 Docker Image(映像) 的倉庫，那今天要介紹的 Docker Registry (註冊表) 就可以把它想像成倉庫的倉庫，那究竟倉庫的倉庫是什麼意思呢？

## 先複習 Docker Repository（儲存庫）

我們可以把多個不同版本和標籤的 Docker Image(映像)，放在 Docker Repository（儲存庫）這個倉庫保存著，每個 Docker 倉庫都只能有一個名稱，但可能有一個或多個 Tag (標籤) ，用來辨別 Image(映像) ，這也就是為什麼我們前一章使用 `docker images` 指令時，出現的是這些資訊

![Docker Repository](/image/dockerDay8/8_1.png)

所以像是 `myapp` 以及 `ruby` 都是不同 Docker Repository（儲存庫），但是 `myapp` 這個 Docker Repository（儲存庫）可以有如 `latest` 與 `v1` 一個或多個 Tag (標籤)，所以我們可以用 `myapp:v1` 來辨別要找哪個 Image(映像)。

## 那 Docker Registry (註冊表) 倉庫的倉庫又是啥？

直接講結論就是，一個 Docker Registry (註冊表)可以包含多個 Docker Repository（儲存庫），每個 Docker Repository（儲存庫）可以包含多個不同版本的 Docker Image(映像) 。

## Docker Hub 是 Docker Registry (註冊表)，也是 Docker Repository（儲存庫）？

我鑽牛角尖的個性又出來了，前面明明說 Docker Hub 是 Docker Repository（儲存庫）的一種公開官方平台，現在為什麼說既是 Registry (註冊表)又是 Repository（儲存庫）？搞得我好亂啊！
舉個例子來說：
Docker Hub 是一個公共的 Docker Registry (註冊表) ，而在 Docker Hub 裡面，可以有很多個的 Docker Repository（儲存庫），例如我們前面範例用到的 [ruby](https://hub.docker.com/_/ruby) 是其中一個 Docker Repository（儲存庫），這個 Docker Repository（儲存庫）可能包含與 Ruby 相關的多個不同版本的 Docker Image(映像) ，如上面的：3.2.2, latest 等等版本。我們可以使用 docker pull 命令來下載這些 Image(映像)，並使用它。

在這篇文我們清楚了解到 Docker Registry (註冊表) ，是一個中央存儲和分發 Docker Image(映像) 的服務，那明天我們繼續了解，出鏡率極高的 Docker Image(映像) 吧。
