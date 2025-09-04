---
title: Docker - 在 EC2 instance 加上 docker compose 跑起來
tags:
  - Docker
  - 部署
  - EC2
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-07 20:17:18
---

昨天我們實際讓 EC2 instance 使用 pull 下來的 image 啟動 container，因為遇到熟悉的錯誤，所以今天要來在 EC2 instance 加上 docker-compose.yml 跑起來。

## 建立 docker-compose.yml 檔案，並寫入資料

在終端機要建立、編輯檔案可以選擇自己熟悉的文本編輯器，我這邊是使用 `vim` 編輯器，若是對於一些 vim 編輯器的語法有興趣深入鑽研，可以另外再去找文章學習，這邊只會簡單介紹基礎 vim 語法。

### 1. 在對的路徑下創建、編輯 docker-compose.yml

一樣先進入到 EC2 instance 的終端機，使用 vim 指令建立加進去檔案編輯

```vim
vim docker-compose.yml
```

![用 vim 建立 docker-compose.yml](/image/dockerDay22/22_1.webp)

這串按下 enter 會出現如下的編輯畫面
![vim 編輯器](/image/dockerDay22/22_2.webp)

### 2. 寫入對的資料到 docker-compose.yml

我們可以按鍵盤 `i` 鍵來切換成輸入模式，並將本來專案的 docker-compose.yml 檔內容給貼上
❗ 特別注意 ❗
本來在本機我們是根據本機的 dockerfile 來建置 app 服務的容器(如下)

```docker
app:
  build:
    context: .
```

但這邊我們要改使用我們在 Docker Hub 拉下來的 image ，來建立 app 容器，所以需要改成(如下)

```docker
app:
  image: krystallll/docker_test:1.0
```

如此一來現在 `docker-compose.yml` 內容會長成如下：
![docker-compose.yml](/image/dockerDay22/22_3.webp)

### 3. 儲存並退出 vim

剛剛使用鍵盤 `i` 進入輸入模式，當需要退出輸入模式時，可以使用 `esc` 鍵，這時試試如何敲打鍵盤都不會有字被打出來，這時我們需要儲存檔案並退出檔案，可以使用 `:wq` 鍵，是 `儲存並退出` 的意思。
![儲存並退出 vim](/image/dockerDay22/22_4.webp)

## docker-compose up 試試

順利退出來後我們可以試跑看看 docker-compose

![docker-compose up](/image/dockerDay22/22_5.webp)

> -bash: docker-compose: command not found

看到這個意思代表他連 docker-compose 都沒有

## 安裝 docker-compose

那就來找找應該如何安裝 docker-compose 吧！谷歌大神我又來了，我下關鍵字 `Amazon Linux install Docker Compose` 找到一篇 [Stack Overflow](https://stackoverflow.com/questions/63708035/installing-docker-compose-on-amazon-ec2-linux-2-9kb-docker-compose-file) 的文章
看起來安裝 docker-compose 可以試試這個步驟，步驟內容大概是：

1. 安裝 docker-compose
2. 調整檔案權限
3. 確認 docker-compose 版本，以確認是否安裝成功

```
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose version
```

我根據步驟實作，也確實安裝成功 v2.22.0 版本的 docker-compose
![安裝 docker-compose](/image/dockerDay22/22_6.webp)
![確認 docker-compose 版本](/image/dockerDay22/22_7.webp)

## 重新 docker-compose up

這時我們重新 docker-compose up 試試，可以看到容器有順利啟動了
![docker-compose up](/image/dockerDay22/22_8.webp)

## 畫面檢查

一樣回到瀏覽器，查詢網址，可以看到當初熟悉的錯誤訊息，要我們 create database，我們一樣可以選擇點按瀏覽器的按鈕

![查詢網址](/image/dockerDay22/22_9.webp)

這邊一樣需要 `db:migrate` ，那我就來教大家另一個不按按鈕進到 docker 容器的語法

![db:migrate](/image/dockerDay22/22_10.webp)

首先我們需要先進到 app 這個容器裡面，為了沁去容器我需要先知道這個容器的名稱，使用最常用的

```docker
docker ps
```

![docker ps](/image/dockerDay22/22_11.webp)

可以看到容器名稱為 `ec2-user-app-1` ，那我就可以使用以下語法進到容器內，

```docker
docker exec -it ec2-user-app-1 sh
```

![進到容器內](/image/dockerDay22/22_12.webp)

可以看到我們順利進到 app 資料夾裡，也就是我們的 rails ，這時我們就可以順利在這邊下

```rails
rails db:migrate
```

![rails db:migrate](/image/dockerDay22/22_13.webp)

回到網頁重整一下，就可以看到我們心心念念的空白首頁 🥳🥳🥳

![重整網頁](/image/dockerDay22/22_14.webp)

## 試試用其他人身份搜尋網址

但謹慎的我還是想多確認一下，我是不是真的有部署成功，這時可以使用其他身份搜尋看看，我使用 Chrome 的無痕模式

![無痕搜尋](/image/dockerDay22/22_15.webp)

一樣輸入網址，成功收工！！！😍😍😍
![成功顯示網址](/image/dockerDay22/22_16.webp)
