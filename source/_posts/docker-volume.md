---
title: Docker - 使用 Volumes 讓我的資料庫保存之前的資料
tags:
  - Docker
  - 部署
  - volume
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-13 23:13:24
---

已經部署完成了~~我們現在就混水摸魚等鐵人賽 30 天到(開玩笑的)~~ ，今天還有什麼好介紹的呢？其實是我昨天發現了一個部署的大 bug ，就是當我每次 `docker-compose down` 後，再次 `docker-compose up` 都會需要重新 create db 等於我每次的資料都沒留下來~~基本上比本機還不如~~😅
那今天就立馬來解決這個 bug ！我首先先打開我久違的專案 (終於暫時脫離 AWS EC2) ，一想到 docker 裡跟 database 有關的設定，我就先去 docker-compose.yml 的 postgresql 服務裡看起：

```docker
db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
```

於是我下關鍵字 `docker-compose db save data` 問最愛的谷歌大神，找到第一個搜尋結果 [stackoverflow](https://stackoverflow.com/questions/38793821/docker-compose-how-to-store-database-data) ，就看到疑似答案的東西，就是需要加上 volumes 。

## Volumes 是做什麼的？

其實在 [DAY 11 - 拆解 Dockerfile 的關鍵字(下)](https://ithelp.ithome.com.tw/articles/10327820) 就有稍微介紹過，但我們今天既然要使用就來詳細介紹一下吧！

> 用於將容器內的檔案或目錄與 Docker 主機上的檔案或目錄進行關聯的設定。這可讓您在容器之間或容器與主機之間共用檔案和資料。

上面解釋讓人一知半解，下面直接翻成白話：

因為我們每一個容器的生命週期根據 docker-compose up 開始，由 docker-compose down 結束，當這個容器停止並刪除時，就算我們再次 docker-compose up 也只是啟動新的容器，跟前一個啟動的容器**並無關聯**。

所以說當像是 database 這種需要儲存資料的東西，我們就會需要有一個，不受容器生命週期影響的地方存放他，那這就是使用 Docker 中的 volumes 的用途。

我們可以使用 volumes 將資料存到主機上，如此一來他就不會受容器的生命週期影響，並且可以將容器內部執行的操作寫入 volumes，以便資料的儲存。

## 如何使用 volumes？

```docker
volumes:
  - /主機路径:/容器路径
```

我們根據上面查到的 [stackoverflow](https://stackoverflow.com/questions/38793821/docker-compose-how-to-store-database-data) 可以知道容器內的路徑是 `/var/lib/postgresql/data`

所以我就可以將 db 加上

```docker
volumes:
   - database:/var/lib/postgresql/data
```

📍 主機路徑 `database` 可以自己規劃，因為如果那個路徑不存在 Docker 會自己建立，所以也可以寫成。

並且在下面定義 database 這個 volumes

```docker
volumes:
  database:
```

現在 `docker-compose.yml` 長成

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
    volumes:
      - database:/var/lib/postgresql/data

volumes:
  database:
```

## 確認是否儲存資料

重新 `docker-compose up` ，並且 `rails db:create` 加上 `rails db:migrate` 後，我們先隨便新增一個 user 看到新增成功後

![新增一個 user](/image/dockerDay28/28_1.webp)

再 `docker-compose down` 移除容器後，重新 `docker-compose up` 確認剛剛新增的 user 是否存在

![確認剛剛新增的 user 是否存在](/image/dockerDay28/28_2.webp)

看到他存在就代表成功了，那 EC2 instance 上面的 docker-compose.yml 也是相同作法，這邊就交給大家自己實做了，那今天就到這邊～
