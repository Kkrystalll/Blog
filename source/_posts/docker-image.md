---
title: Docker - Docker Image(映像) 用模型烤出一個個一樣的雞蛋糕
tags:
  - Docker
  - 部署
  - image
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
date: 2023-09-24 02:22:12
---

Docker Image(映像) 的特性是輕量且獨立，其中包含了運行應用程式所需的代碼、工具、資料庫和設置等等。Docker Image(映像) 不但方便移植，且封裝應用程式，還有他們互相依賴的關係，我們可以在不同的環境中部署和運行 Docker Image(映像) ，對我來說最大的優點是，不用擔心環境差異或配置的問題。

## Image(映像) 的特性

### 不可變性：

Docker Image(映像) 是不可變的，一旦構建完成，就不能更改。若我們想要更改時，就需要建一個新的 Image(映像)，
這個特性的 **優點**

1. 保證了 Image(映像) 在不同環境中的一致性
2. 因為每次都是建立新的 Image(映像) ，所以方便版本控制

當然也有 **缺點**

1. 若是一直需要建立新的 Image(映像)，可能就會需要更多的存儲空間
2. 建立 Image(映像) 的時間，可能會延長部署的時間

### 分層結構：

Docker Image(映像) 是項千層蛋糕一樣一層一層的建立的。每層都包含了文件系統的一部分，以及對上一層分層的修改。
**優點**

1. 這使 Image(映像) 可以共享和重用相同的分層，所以可以節省存儲空間
2. 當我們要部署時，只需要傳輸新的分層，而不用從頭傳輸，可以有效減少傳輸時間

**缺點**
也因為層層相疊的影響，所以如果有層存在安全漏洞，那後面相依的層都會有相同的安全問題

總結 Docker Image(映像) 的主要優勢包括 **易於分發** 、 **跨平台** 、 **快速部署** 和 **版本控制** 。

## Docker Image(映像) 的常用語法

### 查看本地的所有 Image(映像)

```docker
docker images
```

![看本地的所有 Image](/image/dockerDay9/9_1.webp)

這邊可以看到我前面幾天建立的 Image(映像)

### 拉取 Image(映像)

```docker
docker pull <映像名稱:版本>
```

### 刪除 Image(映像)

```docker
docker rmi <映像ID或名稱>
```

📍 `rmi` 就是 remove image 的縮寫

### 建立 Image(映像)

```docker
docker build -t <映像名稱:版本> <Dockerfile路徑>
```

📍 這邊 `-t` 是 tag 的意思
📍 Dockerfile 正常都會位於當前目錄，所以若是當前目錄我們可以寫成

```docker
docker build -t <映像名稱:版本> .
```

`.`就代表當前目錄 ; 需要特別寫路徑，則是因為 Dockerfile 不在當前目錄

### 將 Image(映像) 推送到註冊表

```docker
docker push <映像名稱:版本>
```

這邊的話不特別寫要推到哪個註冊表，就是預設推到 Docker Hub 註冊表
如果有其他註冊表如： Google Container Registry（GCR）或 Amazon Elastic Container Registry（ECR）那就可以寫成

```docker
docker push <目標註冊表URL>/<圖片名稱:版本>
```

### 查看 Image(映像) 的詳細資料

```docker
docker inspect <映像名稱或ID>
```

docker inspect 呈現的方式為 JSON 格式，docker inspect 可以用在**容器**也可以用在**映像**，所有跟這個容器或是映像有關的資訊，都會在這裡記錄。

![docker inspect](/image/dockerDay9/9_2.webp)

(上面圖片只先擷取一部份，實際上有更多資訊喔！)

## 如何生成 Docker Image(映像)?

我已經瞭解了 Docker Image(映像) 這麼多事，但最重要的事，我要怎麼生成屬於我的 Docker Image(映像) 呢？

### Dockerfile

我們要建立 Docker Image(映像) 要先建一個名為 Dockerfile 檔案。Dockerfile 包含了一系列的指令，用於定義如何建立一個鏡像。其中包括選擇基礎鏡像、安裝相依軟件、設置環境變量等。

所以明天我們就要來了解一下 Dockerfile 的關鍵字，以幫助後續我們要撰寫時，能更快上手！
