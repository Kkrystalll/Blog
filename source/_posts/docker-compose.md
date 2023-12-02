---
title: Docker - Docker Compose 讓多個容器同時跑起來
tags:
  - Docker
  - 部署
  - image
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-30 18:07:03
---

前三天不小心陷入 Dockerfile 的除錯地獄中 🤢，甚至最後留下的畫面都還是紅畫面，像極了在一片黑的洞穴中，找不到任何一絲光線的出口，剛開始學 docker 很容易因為遇到這種狀況所以就放棄了。所以今天我們先轉換一下口味，脫離除錯地獄，轉換心情來學習 Docker Compose 是什麼，當然這也是我之後要拿來解決昨天紅畫面錯誤訊息的解決方案。

## Docker Compose 是什麼？

> Docker Compose 是一個用於定義和執行多容器 Docker 應用程式的工具。它允許使用單一 YAML 檔案來設定應用程式的服務、網路和磁碟區，然後使用一個命令來啟動整個應用程式的環境。簡化多容器應用程式的管理和部署，並確保它們在不同環境中的一致運作。

上面的解釋一如既往，都是中文但是接在一起就看不太懂 🤯，這邊就將它翻成白話來解釋，以我們這次範例的專案來說是使用 ruby 這個程式語言，並使用 rails 這個開發框架，資料庫則是使用 PostgreSQL ，上次我們寫在 dockerfile 是寫 `FROM ruby:3.2.2` 最後跑 rails s 這個指令，當這個容器跑起來時，這就是一個簡單的 Ruby 開發環境，可用於運行 Ruby on Rails 應用程式的容器，但是 PostgreSQL 資料庫並沒有被跑起來，所以這時候我們就會需要一個可以執行多個容器(一個運行 Ruby on Rails 應用程式，一個 PostgreSQL 資料庫)的地方，而 Docker Compose 就是這樣的存在。

## 為何我要用 Docker Compose ？

### 優點

執行多個容器：使用 Docker Compose 可以將應用程式的各個部分在不同的容器中運行。包含 Web 伺服器、資料庫等等。可以在一個 YAML 檔案中定義這些服務以及它們之間的關係。

簡化開發環境：Docker Compose 可以在開發過程中模擬生產環境，使開發人員可以在自己的本機上執行多容器應用程式。可以確保開發和測試在一致的環境中進行。

易於使用：Docker Compose 提供了一個簡單的命令列介面，讓您可以輕鬆啟動、停止、重建和管理多容器應用程式。您可以使用單一命令來處理整個應用程式的生命週期。

環境變數和磁碟區允許：Docker Compose 也可以輕鬆設定環境變數，以自訂每個服務的配置。

總之，Docker Compose 是一個特別適用於開發、測試和部署多容器 Docker 應用程式，並有助於簡化容器化應用程式的管理和配置。

## Docker Compose 常見指令

### 啟動容器

```docker
docker-compose up
```

啟動應用程式的所有容器。如果沒有這個容器 Compose 會先建立容器，再啟動它。

### 在背景中啟動容器

```docker
docker-compose up -d
```

在背景中啟動容器，並持續運行。

### 查看容器日誌

```docker
docker-compose logs
```

查看應用程式的 logs。

### 重新建立容器

```docker
docker-compose up --build
```

重新建立(build)應用程式的容器。如果 Dockerfile 或 Compose 檔案有更改，就會用到這個指令。

### 查看容器狀態

```docker
docker-compose ps
```

查看正在運行的容器的狀態，其實跟 docker ps 很類似，只是這是 compose 版本的。

### 停止容器

```docker
docker-compose down
```

停止應用程式的所有容器，並刪除它們的容器、網路等。

### 關閉容器並刪除

```docker
docker-compose down -v
```

`-v` 是 `--volumes` 的縮寫，通常在需要完全重建和清理應用程式環境時很有用。一般的 `docker-compose down` 只會停止和刪除容器、網路等，而不刪除 volumes ，所以若需要刪除 volumes 就可以加上 `-v`。

### 重新啟動容器

```docker
docker-compose restart
```

通常在修改了 Docker Compose 文件時，會用到重新啟動應用程式的容器。

今天已經大約了解 Docker Compose ，明天我們就要來實際寫 Docker Compose 的 YAML 檔案，來解決上次撰寫 Dockerfile 並跑起來得容器，所留下的紅畫面。
