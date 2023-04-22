---
title: Docker - 一起認識 Docker
date: 2023-04-22 18:31:03
comments: false
toc: true
cover: "/image/docker.png"
categories:
  - Docker
  - 部署
tags:
  - Docker
  - 部署
---

因為工作上剛好需要將專案自動部署，所以接觸了 Docker ，所以就決定用文章來讓大家一起了解為何我需要用 Docker ，以及介紹一些基礎語法。

## Docker 是什麼？

Docker 是一種軟體平台，他使用 Google 公司推出的 Go 語言實作，可讓您快速地建立、測試和部署應用程式。

## Docker 想解決的問題

改善傳統虛擬機器因為需要額外安裝作業系統，導致啟動慢、佔較大記憶體的問題。
以我工作上寫 rails 專案為例，所有需要共同開發的人，都至少需要先裝好 ruby 、 rails 、 資料庫(我們使用 PostgreSQL )，但是當今天我將專案包好 Docker ，就算我電腦沒有先安裝好以上項目，也可以使用簡單的 `docker compose up` 指令，使用 docker 打開我的專案，那絕對是可以省下安裝的時間與記憶體容量。

---

當我們已經了解 Docker 能做到哪些事後，就立刻來認識使用 Docker 時，必須知道的 Docker 三寶吧！

## Docker 三寶

### 一. Image(映像檔)

我們可以把它想像成一個烤雞蛋糕模型，而這個模型可以重複考出新的、一樣的雞蛋糕 (container)，正常我們說把專案包 Docker ，可以想像成把專案做成一個雞蛋糕模型，當我們有固定的模型後，可以將其拿來重複使用。
![鯨魚烤模](https://thumbnail7.coupangcdn.com/thumbnails/remote/492x492ex/image/retail/images/2020/09/16/17/1/027a470c-46cc-4695-a615-28f68b5a3cc7.jpg)
圖片出處：[coupang](https://www.tw.coupang.com/products/%E8%BF%B7%E4%BD%A0%E9%AF%A8%E9%AD%9A%E9%AF%8A%E9%AD%9A%E8%9B%8B%E7%B3%95%E6%A8%A1%E5%9E%8B-4%E5%85%A5-21002118279854)

至於可以重複使用的 image ，可以從兩種方式獲得：

1. 可以從遠端 pull 下包好的 image ， [如：pg](https://hub.docker.com/_/postgres) 來重複利用
2. 也可以自己包屬於自己專案的 image 在公司內部做使用

### 二. Container(容器)

如果說 Image 是雞蛋糕的烤模，那 Container 就可以比喻成，用模型 (Image) 做出來的實體「雞蛋糕本人」，而這個實體可以被 **啟動**、**開始**、**停止**、**刪除** ，且每個容器之間都相互隔離，不會互相污染，就像是烤出來的雞蛋糕都是獨立個體。
![鯨魚雞蛋糕](https://i.imgur.com/8y3nc4c.jpg)
圖片出處：[扮公伙](http://houseplayhouse.blogspot.com/)

### 三. Repository(倉庫)

當我們製作出可以重複使用的烤模 (Image) ，想必是需要一個可以存放他們的倉庫 (Repository) ，我們可以將 Image 推送到遠端倉庫 (Repository) ，需要時再到遠端倉庫 (Repository) 將其 pull 下來使用即可。
![蛋糕置涼架](https://img.alicdn.com/i1/4122897559/O1CN01ho0Oq125i4YHvW6FU_!!0-item_pic.jpg_q50s50.jpg)
圖片出處：[淘寶](https://world.taobao.com/product/%E8%9B%8B%E7%B3%95%E6%A8%A1%E5%85%B7%E6%94%B6%E7%B4%8D.htm?)

而較常使用的遠端倉庫有 [Docker Hub](https://hub.docker.com/) 或 AWS 等。

那今天我們對於 Docker 有初步的認識了，接下來在繼續學習 Docker 時，也會更清楚其運作的方式。
