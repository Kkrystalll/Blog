---
title: Docker - 在我部署的專案使用 Traefik 取得 HTTPS 協定(一)
tags:
  - Docker
  - 部署
  - Traefik
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
  - Traefik
date: 2023-10-09 19:27:10
---

我參考 [官網](https://doc.traefik.io/traefik/getting-started/quick-start/) 用 Docker 啟動 Traefik ，跟者步驟試試

## 在 docker-compose.yml 加上 traefik 服務

### 1. 先將原本的 docker-compose 停下

```docker
docker-compose down
```

![Traefik 配置](/image/dockerDay24/24_1.webp)

確認一下有沒有容器在跑

```docker
docker ps
```

![Traefik 配置](/image/dockerDay24/24_2.webp)

### 編輯 docker-compose.yml

官網上面說在 docker-compose.yml 加上 reverse-proxy 這個服務

```docker
reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:v2.10
    # Enables the web UI and tells Traefik to listen to docker
    command: --api.insecure=true --providers.docker
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
```

我根據上方官網的提示將 service `reverse-proxy` 改為 `traefik` ，然後重新啟動 docker-compose 一次，首先使用 vi 來編輯:

```docker
vi docker-compose.yml
```

現在的 docker-compose.yml 長這樣：

```docker
version: "3.9"
services:
  app:
    image: krystallll/docker_test:1.0
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
  traefik:
    image: traefik:v2.10
    command: --api.insecure=true --providers.docker
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

### 重新啟動容器

```docker
docker-compose up --build
```

![Traefik 配置](/image/dockerDay24/24_3.webp)

### 去看 Traefik 的 API 原始資料

官網說可以去以下網址看 Traefik 的 API 原始資料

```
http://localhost:8080/api/rawdata
```

![Traefik 配置](/image/dockerDay24/24_4.webp)

### 開 EC2 8080 port

看到無法連上網站，我先盲猜是因為沒有開 8080 port，所以跟之前一樣我先去 AWS EC2 裡跟之前設定 3000 port 一樣去設定 8080 port。

一樣先去編輯 Security Groups

![Traefik 配置](/image/dockerDay24/24_5.webp)

加上 8080 port

![Traefik 配置](/image/dockerDay24/24_6.webp)

### 重看瀏覽器 Traefik 的 API 原始資料

重新看

```
http://localhost:8080/api/rawdata
```

![Traefik 配置](/image/dockerDay24/24_4.webp)

一樣是無法連上網站，突然靈機一動上面的網址是 `http://localhost` 但現在網址是 `http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com` 所以我換網址為 `http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata` 可以看到以下

![Traefik 配置](/image/dockerDay24/24_8.webp)

終於成功看到 API 原始資料，那明天我們繼續來完成使用 Traefik 取得 HTTPS 協定。
