---
title: 撰寫 Docker Compose 的 YAML 檔案(下)
tags:
  - Docker
  - 部署
  - docker-compose
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-10-02 20:06:35
---

昨天將 app 服務 docker-compose.yml 的部分撰寫好了，今天就來寫資料庫服務的部分

## 將資料庫服務定義好

### 定義 PostgreSQL 服務名稱

跟昨天一樣取一個資料庫在 compose 裡的唯一名稱

```docker
db:
```

為了簡明扼要又好讀，就直接叫做 `db`

### 根據什麼建立容器？

昨天我們使用

```docker
build:
   context: .
```

是因為我們希望可以根據本地的 Dockerfile 來建置容器 ; 但今天我們的資料庫就是因為沒有在 Dockerfile 建立出來的容器裡，所以我們應該要找一個符合我安裝的 PostgreSQL 版本的 image 寫成：

```docker
image: <我要引用的 image>
```

#### 確認 PostgreSQL 版本

首先若是本機已經安裝 PostgreSQL 可以在終端機下

```
psql
```

使用 psql 指令連接到您的 PostgreSQL 資料庫，可以看到如下

![psql](/image/dockerDay17/17_1.png)

這邊可以看到後面的 14.5 就是 PostgreSQL 服務的版本，但若是你還是想再確認一次，或是知道更多詳細訊息，也可以在這邊再下

```SQL
SELECT version();
```

![psql 詳細訊息](/image/dockerDay17/17_2.png)
![psql 詳細訊息](/image/dockerDay17/17_3.png)

如圖我們就可以知道有關版本的更多詳細訊息。

#### 在 Docker Hub 找適合的 image

要找公開包好的 image ，第一時間想到 Docker Hub，我在 Docker Hub 裡搜尋 [postgres](https://hub.docker.com/_/postgres) 找尋版本 14 開頭的 tag

![Docker Hub postgres](/image/dockerDay17/17_4.png)

因為沒有完全符合 14.5 的 tag ，所以我就選擇 `14-alpine` ，這時 docker-compose.yml 可以寫成：

```docker
image: postgres:14-alpine
```

### 完成 db 服務設定

剛剛找好根據哪個 image ，然後呢？還有哪些要定義的？我們可以順勢參考頁面 [postgres](https://hub.docker.com/_/postgres)

![Docker Hub postgres](/image/dockerDay17/17_5.png)

這邊寫到需要設置環境變數，我們就加上去

```docker
environment:
   POSTGRES_PASSWORD: password
```

倒是有一個 `restart: always` 好像是先前沒有看過的

### docker compose 的 restart 是什麼？

如其名就是重新開啟，再解釋清楚一點就是容器在什麼狀況下需要重新 restart ，通常用於確保容器在故障或異常情況下能夠恢復正常運行。
而 restart 有多種選項：

#### no(預設值)

如果容器停止不會自動重新啟動容器，若希望容器啟動，需要手動啟動。

#### always

設定為 always，那無論手動停止 `docker-compose stop` 容器，還是將服務停止 `docker-compose down`，容器都會在停止後自動重新啟動。這對於保持應用程式的可用性非常有用，因為它確保了即使發生故障，容器也可以重新啟動。

#### on-failure

表示當容器因為失敗而停止時，才會自動重新啟動。如果容器正常退出（我自己退出的），將不會自動重新啟動。

#### unless-stopped

我將它理解為跟 always 九成像，唯一一成不同則是，如果手動停止容器，並 **不** 會自動重新啟動。

這邊根據我的狀況我希望加上

```docker
restart: on-failure
```

並且也順便幫 app 服務也一併加上。

這時我的 `docker-compose.yml` 長成如下：

```docker
version: "3.9"
services:
  app:
    build:
      context: .
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_HOST: db
    restart: on-failure
  db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
```

## 啟動 docker-compose 看看

```docker
docker-compose up --build
```

使用 `docker-compose up --build` 建立＋啟動容器，通常會需要一小段時間，我們先讓子彈飛一會兒...

![建立＋啟動容器](/image/dockerDay17/17_6.png)

容器看似開起來，有動靜

![建立＋啟動容器](/image/dockerDay17/17_7.png)

但當我打開瀏覽器在網址搜尋 `http://0.0.0.0:3000` ，出現了跟之前一樣的

![瀏覽器頁面](/image/dockerDay17/17_8.png)

記得上次是因為我沒有指定 port ，於是我詢問谷歌大神，搜尋關鍵字 `docker compose port` 可以看到第一篇文章，也就是 [官方文件](https://docs.docker.com/compose/networking/) 說明 docker-compose.yml 檔也要寫 port ，這樣才可以知道容器的 port 要對應到主機的哪個 port ，供主機訪問，所以我在 app 的服務中加上

```docker
ports:
   - 3000:3000
```

現在的 `docker-compose.yml` 長成如下：

```docker
version: "3.9"
services:
  app:
    build:
      context: .
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_HOST: db
      POSTGRES_PORT: 5432
    restart: on-failure
    ports:
      - 3000:3000
  db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
```

要重新將容器 build ，需要先將 compose 內的容器移除

```docker
docker compose down
```

![移除容器](/image/dockerDay17/17_9.png)

再重複上述動作

```docker
docker-compose up --build
```

這時我們打開瀏覽器在網址搜尋 `http://0.0.0.0:3000` 可以看到

![Create database](/image/dockerDay17/17_10.png)

仔細看錯誤訊息只是因為我們的 database 還沒有建立，所以直接點選上面的 `Create database` 按鈕，這個按鈕其實就是我們平常打的 `bin/rails db:create` 。

順利 Create database 後，不曉得大家腦中有沒有預期到等等會發生的錯誤了？

![rails db:migrate](/image/dockerDay17/17_11.png)

就是還需要跑 `rails db:migrate` ，一樣先點按按鈕即可

![成功開啟專案](/image/dockerDay17/17_12.png)

終於 🥳🥳🥳 看到了我們空空如也的首頁，這代表以後我們都可以透過 `docker compose up` 來啟動容器！
