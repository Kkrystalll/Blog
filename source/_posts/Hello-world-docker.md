---
title: Docker - 你好世界 Hello World
tags:
  - Docker
  - 部署
  - Hello,world!
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-17 17:31:03
---

昨天我們安裝了環境，但我們要如何確保電腦可以跑 Docker 了呢？這時候就要來執行學習程式語言的起手式 Hello World ，在 Docker 中只要成功跑 Hello World 就是代表成功安裝。
但在這之前我們需要先知道 Docker 架構，以及學習 Docker 必用的三寶。

## Docker 基礎架構

根據 [官方文件](https://docs.docker.com/get-started/overview/) ，可以得知 Docker 基礎架構分為三大類

### Docker client(Docker 客戶端)

其實看中文大概可以理解為客戶的用戶界面，也就是說我們可以透過指令或操作，來控制和管理 Docker 容器和鏡像。

### Docker daemon(Docker 守護進程)

是在後台運行的核心元件，負責管理和監控 Docker 容器、鏡像、網絡和其他相關的資源。

### Docker API

Docker API 遵循 RESTful 設計，允許外部應用程式和工具與 Docker 守護進程進行通訊和互動。通過 API，開發人員可以使用不同的程式語言和工具來管理和控制 Docker 容器、鏡像、網絡等相關資源。

![Docker API](/image/dockerDay2/2_1.jpg)

以上結合起來舉個例子就是，我們平時是在 Docker client 下的指令，如：`docker run hello-world` ，當我們打了這句指令後， Docker client 會將這個命令轉換為一個對 Docker API 的 HTTP 請求，而這個 HTTP 請求會被發送到正在運行的 Docker daemon，進而由 Docker daemon 來運行我們下的指令。

## Docker 三寶

### 一. Image(映像檔)

我們可以把 Image(映像檔) 想像成一個烤雞蛋糕模型，而這個模型可以重複考出新的、一樣的雞蛋糕 (container)，正常我們說把專案包 Docker ，可以想像成把專案做成一個雞蛋糕模型，當我們有固定的模型後，可以將其拿來重複使用。 Image(映像檔) 是唯讀的，其中包含了應用程序運行所需的所有內容，包括代碼、環境、相依性等。
![鯨魚烤模](/image/dockerDay2/2_2.jpeg)

圖片出處：[coupang](https://www.tw.coupang.com/products/%E8%BF%B7%E4%BD%A0%E9%AF%A8%E9%AD%9A%E9%AF%8A%E9%AD%9A%E8%9B%8B%E7%B3%95%E6%A8%A1%E5%9E%8B-4%E5%85%A5-21002118279854)

至於可以重複使用的 image ，可以從兩種方式獲得：

1. 可以從公開 Docker Hub (遠端) pull 下包好的 image ， [如：pg](https://hub.docker.com/_/postgres) 來重複利用
2. 也可以自己包屬於自己專案的 image 在公司內部私有的 Repository(倉庫) 做使用

### 二. Container(容器)

如果說 Image 是雞蛋糕的烤模，那 Container 就可以比喻成，用模型 (Image) 做出來的實體「雞蛋糕本人」，而這個實體可以被 **啟動**、**開始**、**停止**、**刪除** ，且每個容器之間都相互隔離，不會互相污染，就像是烤出來的雞蛋糕都是獨立個體。
![鯨魚雞蛋糕](/image/dockerDay2/2_3.jpeg)

圖片出處：[扮公伙](http://houseplayhouse.blogspot.com/)

### 三. Repository(儲存庫)

當我們製作出可以重複使用的烤模 (Image) ，想必是需要一個可以存放他們的倉庫 (Repository) ，我們可以將 Image 推送到遠端公開或私有的倉庫 (Repository) ，需要時再到遠端倉庫 (Repository) 將其 pull 下來使用即可。
![蛋糕置涼架](/image/dockerDay2/2_4.jpeg)

圖片出處：[淘寶](https://world.taobao.com/product/%E8%9B%8B%E7%B3%95%E6%A8%A1%E5%85%B7%E6%94%B6%E7%B4%8D.htm?)

而較常使用的遠端倉庫有 [Docker Hub](https://hub.docker.com/) 或 AWS 等。

大概知道這些概念後就可以來來跑第一個 Hello World 環境。

## 第一次執行 Docker 的 Hello World

前面有提到，我們需要包好的 image 時，可以去公開的 Repository (倉庫)找找，所以我直接搜尋 [Docker Hub](https://hub.docker.com/)，在上方搜尋 [hello-world](https://hub.docker.com/_/hello-world) ，可以看到

這時右上方寫著我們可以執行

```
docker pull hello-world
```

這個其實跟 git pull 是差不多的觀念，就是把遠端儲存庫的 hello-world 拉取到我電腦本地，當我們拉取下來後即可執行

```
docker run hello-world
```

這時我們便會看到 Hello from Docker!（如下圖）
![拉取 hello-world 並啟動](/image/dockerDay2/2_5.png)

當你也順利看到 Hello from Docker! 則是代表安裝成功啦～恭喜踏入鯨魚世界第一步！

但其實上面兩步驟（先 pull 再 run）可以直接一個動作完成，便是直接

```
docker run hello-world
```

這時心裡滿滿髒話，阿一步可以完成的事，憑什麼叫我兩步驟執行？其實上面才是正常流程，但現在可以直接跳過第一步驟的原因是，當你運行 docker run hello-world 時，Docker 首先會檢查本地是否有名為 "hello-world" 的映像，如果沒有，第二步它會去找 Docker Hub 有沒有名為 "hello-world" 的映像，若有，便會先下載 (pull) 該映像，再執行 (run) 映像，可以根據圖片的 log 得知。
![直接拉取並啟動 hello-world](/image/dockerDay2/2_6.png)

無論你是一步驟成功或是兩步驟成功，只要成功看到

```
 Hello from Docker!
```

都是安裝成功，接著明天我們要來介紹 Docker 基本指令(雖然今天其實有偷跑 pull 跟 run XD)
