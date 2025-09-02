---
title: Docker - 認識及建立 AWS EC2 Instance
tags:
  - Docker
  - 部署
  - AWS
  - EC2
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-04 17:30:12
---

## AWS EC2 是什麼？

AWS EC2 全名 Amazon Elastic Compute Cloud ，根據 [官方文件](https://aws.amazon.com/tw/ec2/)，可以看到介紹

> AWS EC2 提供最廣泛、最深入的運算平台，擁有超過 700 個執行個體，可選擇最新處理器、儲存、聯網、作業系統和購買模型，以協助您最有效地滿足工作負載需求。我們是第一間支援 Intel、AMD 和 Arm 處理器的主要雲端供應商，提供唯一具有隨需 EC2 Mac 執行個體的雲端，以及唯一具有 400 Gbps 以太網路聯網的雲端。我們為機器學習訓練提供了最佳價格效能，同時也是每個雲端推論執行個體的最低成本。在 AWS 上執行的 SAP、高效能運算 (HPC)、機器學習 (ML) 和 Windows 工作負載多過任何其他雲端。

看了一長串還是不太懂，直接再翻白話一點 AWS EC2 允許開發人員租用虛擬運算資源，以便在雲端中運行應用程式，使開發人員能夠輕鬆地建立、啟動和管理虛擬伺服器。

## AWS EC2 的優點

### 彈性伸縮：

以前我們需要空間就是要買硬體，但是在 EC2 我們可以根據專案或個人需求，來決定要開一個或多個 EC2 ，這種彈性可以讓人根據應用程式負載需求的增加或減少計算資源。

### 多種執行個體類型：

AWS EC2 可以選擇不同的虛擬伺服器，滿足不同種類的運算工作。

### 作業系統靈活：

我們可以在 EC2 上執行的 Linux 、 Windows Server 或其他作業系統，不會因為不相容被迫換其他作業系統。

### 網路和安全性：

AWS EC2 可以讓人自訂虛擬雲端（VPC）來管理網路配置，還可以控制執行個體的存取權限，以確保安全性。

### 鏡像管理：

EC2 可以建立自己的 AMI（Amazon Machine Image）來定義 EC2 執行個體的配置，包括網路、應用程式和資料。可以輕鬆地複製和部署多個相同配置的 EC2。

### 首先付費：

上面有說到 EC2 的彈性伸縮特性，定價也是如此，只需負擔實際使用的資源付費。

### 全球效能：

AWS 區域分佈在全球各地，包括美國、歐洲、亞洲、南美洲等地區，使用者可以選擇將自己的 EC2 執行個體部署在靠近的區域，以減少網路延遲。

總結來說：AWS EC2 通常用於各種應用程式和工作負載，包括網站託管、應用程式開發和測試、大數據處理、機器學習、容器化應用程式等。我們這次是使用 EC2 來作為 docker 容器部署，屬於其服務中容器化應用程式的一種。

## 開 AWS EC2 機器的起手式

### 登入 / 註冊 [AWS](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation&language=zh_tw#/start/email)

![註冊/登入 AWS](/image/dockerDay19/19_1.webp)
有帳號直接登入、沒帳號可以直接下方點選 create 註冊

### 找到 EC2 的頁面

1. 可以使用上方搜尋欄
   ![搜尋 EC2](/image/dockerDay19/19_2.webp)

2. 可以使用左上方 Services 按鈕，按字母搜尋
   ![Services EC2](/image/dockerDay19/19_3.webp)

## 建立 AWS EC2 Instance

搜尋後即可以看到我的 EC2 儀表板，下圖箭頭處可以看到我們現在有幾個「正在運行中的」instance，若是要建立 instance 也可以直接點擊黃色框框內的 `Launch instance` 按鈕，接著就開始填寫有關 instance 的一連串設置
![Launch instance](/image/dockerDay19/19_4.webp)

### 1. 填寫 instance 名稱

### 2. 選擇作業系統

預設就是 Linux ，也可以選擇您習慣的作業系統

![選擇作業系統](/image/dockerDay19/19_5.webp)

### 3. 選擇 instance type

這是 EC2 運算、記憶體、儲存和網路資源的類型，我這邊就秉持客家精神，選擇預設免費的 `t2.micro`

### 4. 選擇 & 建立密鑰

正常來說我們需要打帳號密碼來連線到 AWS EC2 ，但若我今天有設定密鑰，這個秘藥就可以讓我們不需要打帳號密碼，直接透過密鑰解密就可以連線到 AWS EC2 ，所以我們現在就直接來建立一把密鑰
![建立密鑰](/image/dockerDay19/19_6.webp)

點選 `create new key pair` 就可以開始填寫相關資訊，完成後按 `create key pair`
![建立密鑰資訊](/image/dockerDay19/19_7.webp)

建立後可以在下載項目看到剛剛取名的 `krystal.pem` 下載到電腦裡，這個就是我們的金鑰，我個人會習慣先將這種密鑰有關的檔案保存好。
![儲存密鑰](/image/dockerDay19/19_8.webp)

這時我們就可以選擇用我們剛剛建立的 krystal 這把密鑰來作為解密的工具了
![選擇新建立的密鑰](/image/dockerDay19/19_9.webp)

### 5. 網路設定

![網路設定](/image/dockerDay19/19_10.webp)
除了預設的選項，我還多勾了 `Allow HTTPS traffic from the internet` 跟`Allow HTTP traffic from the internet`

允許透過 HTTPS 與 HTTP 協定（通常使用連接埠 443 與 80）從網際網路到 EC2 ，這對於設置 Web 伺服器並實現安全的 HTTPS 與 HTTP 通訊非常有用。

📍 這兩個選項必勾，因為我們要在 EC2 執行個體上執行 Web 伺服器，並希望任何人只要輸入網址，可以透過 Web 瀏覽器到我們的網站，如果不勾選這兩個選項，那使用者永遠到不了這個網站。

### 6. 建立 instance

![建立 instance](/image/dockerDay19/19_11.webp)
都設定完成就點選右下角的 `Launch Instance`
![建立成功 notice](/image/dockerDay19/19_12.webp)
可以看到成功建立的 notice，這時我們再點到我的 Instances ，可以看到我成功建立的 `docker_test` instance。
![EC2 instance 列表](/image/dockerDay19/19_13.webp)

那今天成功設置了 AWS EC2 instance ，明天開始我們就要嘗試在這個 instance 上面跑我推到 Docker Hub 上的 image，那就明天見啦～
