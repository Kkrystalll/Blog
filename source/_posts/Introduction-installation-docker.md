---
title: Docker - 認識及安裝 Docker
tags:
  - Docker
  - 部署
  - 安裝
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-16 18:31:03
---

因為工作上剛好需要將專案自動部署，所以接觸了 Docker ，所以就決定用文章來讓大家一起了解為何我需要用 Docker ，以及介紹一些基礎語法。

## Docker 是什麼？

Docker 是一個開源的容器化平台，他使用 Google 公司推出的 Go 語言實作，用於將應用程式、環境和相依套件打包成 **輕量**、**可移植** 的容器。這些容器可以在任何地方運行，包括開發機器、測試環境和生產伺服器，確保應用程式在不同環境中的一致運行。其中每個容器都有自己的文件、環境變數和空間，但它們共享主機系統的操作核心。容器化的優點是**非常高效**且**佔用資源少**。

## Docker 想解決的問題及為何我需要使用 Docker

1. **省下安裝時間及記憶體空間**：改善傳統虛擬機器因為需要額外安裝作業系統，導致啟動慢、佔較大記憶體的問題。
2. **環境一致性**：Docker 可以確保應用在不同環境中的一致運行，從開發到測試到生產環境，減少了由於環境不一致而導致的問題。
3. **快速部署**：Docker 可以快速地創建、啟動和停止容器，部署過程可以更加快速簡易。
4. **容器映像管理**：Docker Hub 提供了大量的共用映像 (Image)，讓我們可以直接使用這些映像 (Image)，也可以根據自己的需求自己製作映像 (Image)。

以我工作上寫 rails 專案為例，所有需要共同開發的人，都至少需要先裝好 ruby 、 rails 、 資料庫(我們使用 PostgreSQL )，但是當今天我將專案包好 Docker ，就算其他人電腦沒有先安裝好以上項目，也可以使用簡單的 `docker run` 或`docker compose up` 指令，使用 docker 打開專案，當不需要額外安裝時，絕對是可以省下安裝的時間與記憶體容量，且因為我們是使用同一個 docker 映像 (Image) 來產生容器 (Container)，所以可以確保所有人都在同個環境下運行專案，最後我需要將我的專案部署時，我可以將我 `docker build` 好的映像 (Image) 推送到遠端倉庫 (Repository) ，並在遠端 `docker run` 或`docker compose up` ，在設定好網路等，便可以快速部署。

在了解 Docker 是什麼之後，就先來將 Docker 安裝在自己本機吧！

## 安裝 Docker Desktop

根據[官方文件](https://docs.docker.com/desktop/) Docker Desktop 是一款適用於 Mac、Linux 或 Windows 環境的應用程式，今天就示範以使用 Mac 電腦以及 Windows 安裝 Docker 為例

![安裝 Docker](/image/dockerDay1/1_1.png)

根據[官方文件](https://docs.docker.com/desktop/)可以看到圖示，點選符合電腦環境的連結

### On Mac

當我們使用 [Mac](https://docs.docker.com/desktop/install/mac-install/) 需要安裝 Docker Desktop 時，首先應該要確認自己的電腦是屬於`Intel 晶片` 還是 `Apple 晶片` ，選擇對應的按鈕
![Mac 安裝 Docker](/image/dockerDay1/1_2.png)
若是不確定自己的電腦屬於哪種晶片，只需要

1. 點擊螢幕左上方的 

2. 選擇`關於這台 Mac`
   ![關於這台 Mac](/image/dockerDay1/1_3.png)

3. 即可以看到自己的電腦是哪種晶片
   ![關於這台 Mac 詳細資料](/image/dockerDay1/1_4.png)

點擊之後會直接進下載項目，根據指示可以打開 Docker Desktop 登入，但下載好不是全部，還需要啟動 Docker Desktop
![啟動 Docker](/image/dockerDay1/1_5.png)

啟動完成即可以看到這台電腦現在的容器 (Container) 列表、映像 (Image) 列表、控制台、網路等等圖形化介面。
![Docker 圖形化介面](/image/dockerDay1/1_6.png)

上圖是我的容器 (Container) 列表，可以看到我現在有一個容器目前是 Exited （執行完畢並退出）的狀態，當然各位剛下載的話就會是空的。

### On Windows

若是使用 Windows 就更簡單一點，不需要檢查電腦是哪個晶片，可以直接一鍵下載
![Windows 安裝 Docker](/image/dockerDay1/1_7.png)

點選下載後可以以看右上角已經有下載中的標示
![Windows 下載](/image/dockerDay1/1_8.png)

接著打開電腦的下載檔案夾，可以看到 Docker Desktop，直接雙擊滑鼠安裝
![下載檔案夾](/image/dockerDay1/1_9.png)

根據指示點選
![根據指示點選](/image/dockerDay1/1_10.png)

會看到正在安裝的進度
![正在安裝的進度](/image/dockerDay1/1_11.png)

當安裝好時會需要將電腦重開機，直接點擊 `close and restart` 按鈕
![restart Docker](/image/dockerDay1/1_12.png)

待電腦重開機完成，我們點選 Docker Desktop ，打開就可以按照指示註冊登入了
![註冊登入 Docker](/image/dockerDay1/1_13.png)

到這邊為止是兩種電腦的安裝方式，接著明天我們要教大家第一次執行 Docker ，也就是學習程式的起手式 Hello World ，但在執行之前我們需要先了解使用 Docker 時，絕對必須知道的 Docker 三寶！
