---
title: Docker - 將 docker-compose.yml 裡的 container 取名
tags:
  - Docker
  - 部署
  - docker-compose
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-15 20:21:57
---

昨天整理了 Dockerfile ，今天換 docker-compose.yml 吧！

## 自訂 container 名稱

不知道大家有沒有注意到．每次 `docker-compose up` 時，看到的畫面

![docker-compose up log](/image/dockerDay30/30_1.webp)

這些是他自動給容器的名稱 `docker_test-db-1` 或 `docker_test-app-1` ，因為我們沒有幫容器取名，所以他就根據 `專案名稱＋服務名稱` 來命名。

但是若你不喜歡這個名稱想自定義名稱也是可以的，只要在服務後加上

```docker
container_name: <你想取的名字>
```

在 `docker-compose up` 時就可以看到

![docker-compose up log](/image/dockerDay30/30_2.webp)

成功將名稱更換了，如此一來我們更可以透過 log 知道應該在哪個容器除錯。

## 結語

這次使用 Docker 部署你的專案就到這邊，在撰寫的時候真的很容易撞牆，不知從何除錯的狀況發生，所以這次的文章，有根據我是如何下關鍵字，以及如何除錯的過程帶著大家一步一步到完成，祝福大家都可以成為 Docker 大師！