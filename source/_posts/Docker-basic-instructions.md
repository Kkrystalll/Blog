---
title: Docker - 基本指令
tags:
  - Docker
  - 部署
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-18 00:31:03
---

昨天我們在 Docker 中成功跑 Hello World ，今天來了解一些 Docker 的基本指令

要知道 Docker 有哪些指令可以打的話，我們可以直接在終端機下

```docker
docker
```

這個指令，這時可以看到所有 docker 可以下的指令
![docker 可以下的指令](/image/dockerDay3/3_1.png)

其中我們可以看到三大分類

### Options（選項）：

是用於設定 Docker 執行環境的全域設定。
例如：

```docker
docker --help
```

可以看到 docker 列表

```docker
docker -v
```

及

```docker
docker --version
```

可以看到現在 docker 的版本

### Management Commands（管理命令）：

這些命令用於管理 Docker 容器、映像、網絡。
例如：`docker container` 命令是用來管理容器 ;
`docker image` 命令是用來管理映像 ;
`docker network` 命令是用來管理網絡。
這些命令能創建、查看、修改和刪除 Docker 資源。

### Commands（命令）：

這些是 Docker 執行的實際命令。
例如：`docker run` 用於運行容器，`docker build` 用於構建映像，`docker-compose up` 用於啟動 Compose 的項目。
這些命令讓你執行特定的操作，例如創建新容器、構建映像、運行服務等。

以昨天打過的 `docker run hello-world` 其實是 `docker container run hello-world` 的縮寫，那將這句話對應到上面，**docker** 是 registry (倉庫) ; **container** 就是 Management Commands（管理命令），要`管理執行的東西`， **run** 就是 Commands（命令），我自己是把他想像成 `動作` 或是 `做什麼？` ，至於 hello-world 就是 image 名稱啦。

接著我來介紹會用到的一些常見指令

```docker
docker container run hello-world
```

或是

```docker
docker run hello-world
```

這個相信前面說明的很清楚，就是使用 hello-world 這個 image ，來執行 Docker container

```docker
docker ps
```

ps 是 "process status" 的縮寫，也就是說這個指令可以列出正在進行的容器，顯示容器的 ID、名稱、狀態、使用的映像等。可以快速查看容器的運行狀態以及相關的資訊。

若是想看到全部，無論是正在進行的容器，或是已經停止的容器我們可以下

```docker
docker ps --all
```

或是他的縮寫

```docker
docker ps -a
```

可以看到全部無論是正在運行的，或是已經停止的 container
![](https://hackmd.io/_uploads/S1f4liJa2.png)

講到停止就順道來介紹一下，我們使用 run 來啟動容器，反之就要可以停止 container 運行

```docker
docker container stop hello-world
```

或是

```docker
docker stop hello-world
```

這兩個指令可以解釋為停止名為 "hello-world" 的 Docker 容器。

講到這些但我們的 hello-world 這個 image 是哪來的呢？

```docker
docker build .
```

在當前的目錄中尋找名為 Dockerfile 的檔案，然後使用這個 Dockerfile 建立一個 Docker 映像（image）。但因為抹沒有特別指定名稱和標籤，所以這個 image 會使用一個隨機生成的 ID 作為名稱。

📍 特別注意「.」就代表 **當前的目錄** 所以千萬不可省！

若是想要建立一個名為 myapp 的 image ，可以使用

```docker
docker build --tag myapp .
```

或是

```docker
docker build -t myapp .
```

這會根據當前的目錄中尋找名為 Dockerfile 的檔案，然後使用這個 Dockerfile 建立一個名為 `myapp` 的 Docker 映像（image）。

當我們建立完自己的 myapp 這個 image 一定會想查看我本地是否有 myapp 這個 image ，這時可以使用

```
docker images
```

可以列出本地的所有 images。

再來兩個語法是，若你會使用 git 版控一定很好理解

```docker
docker pull <imagename>
```

從遠端 registry (倉庫)下載 image (映像)到本地。

```docker
docker push <image>
```

將本地的 image (映像)上傳到遠端 registry (倉庫)。

今天先介紹這些一定會用到的語法，大概可以使用這些語法自己練習拉取，或是看看自己的 docker，之後會根據類別再一一詳細介紹，那就明天見！
