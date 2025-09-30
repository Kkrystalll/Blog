---
title: Docker - 在我部署的專案使用 Traefik 取得 HTTPS 協定(三)
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
date: 2023-10-12 11:07:02
---

昨天將 domain 設定好，確定目前的 `mydocker.online` 是已經跟我們的服務連上的， `docker-compose.yml` 目前長成如下，那我們今天要一步步來除錯。

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
      - 80:3000
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.app.loadbalancer.server.port=3000"
      - "traefik.http.routers.app.rule=Host(`mydocker.online`)"`
      - "traefik.http.routers.app.entrypoints=web"
      - "traefik.http.routers.app.tls=true"
      - "traefik.http.routers.app.tls.certresolver=letsencrypt"
  db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
  # traefik:
    # image: traefik:v2.10
    # command:
      # - "--api.insecure=true"
      # - "--providers.docker=true"
      # - "--providers.docker.exposedbydefault=false"
      # - "--entrypoints.web.address=:80"
      # - "--entrypoints.websecure.address=:443"
      # - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      # - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      # - "--certificatesresolvers.letsencrypt.acme.email=krystal@5xcampus.com"
      # - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    # ports:
      # - "80:80"
      # - "443:443"
      # - "8080:8080"
    # volumes:
      # - "./letsencrypt:/letsencrypt"
      # - /var/run/docker.sock:/var/run/docker.sock
```

當我將上述註解都打開，並且移除 app 服務的 `ports: - 80:3000` (移除是因為 traefik 服務有 80 port ，所以可以由他指揮去 app 服務) 後，重新 `docker-compose up` ，再去網址 `https://mydocker.online` 時，不外乎一樣是 404 page not found。

統整一下我搜集到的可能連接不上的所有原因，來一一檢查修改

## 服務是否有在共同的 Network 下運行？

因為我的 docker-compose.yml 沒有特別設定在哪個網路下執行，所以我們就先來設定一下，確保我的 docker-compose 的所有服務都在同個網路下執行，可以互相通信，如果大家對於 Network 不太熟悉，可以參考 [Docker - Docker Networks(網路)容器與容器間的橋樑](https://kkrystalll.github.io/posts/docker-networks/)

首先我要先建立一個網路，我就取名為 `development` 好了，那就在我的 AWS EC2 instance 終端機裡面，我可以先確認現在的 Network 有哪些

```docker
docker network ls
```

再來建立名為 `development` 的網路

```docker
docker network create development
```

![create network](/image/dockerDay27/27_1.webp)

看到這串 SHA 值就代表建立成功！

接著去 `docker-compose.yml` 加上這個 `development` 的網路

```docker
version: "3.9"
services:
  app:
    image: krystallll/docker_test:1.0
    networks:
      - development
    (...中間略)
  db:
    image: postgres:14-alpine
    networks:
      - development
    (...中間略)
  traefik:
    image: traefik:v2.10
    networks:
      - development
    (...中間略)
networks:
  development:
    external: true
```

這邊只有我加上網路的片段，其他都跟本來一樣就先省略。

## 是否有開啟安全群組？

接者每當我們前面要去 port 3000 或是 8080 我們都有檢查 Security Groups ，但是現在我們在 docker-compose.yml 的 traefik 服務，是開了三個 port 80、443、8080

```docker
ports:
   - "80:80"
   - "443:443"
   - "8080:8080"
```

所以現在要去檢查 Security Groups 是否都有開？

![檢查 Security Groups](/image/dockerDay27/27_2.webp)

發現少了 https 的 443 port，所以我加開了 443 port 的 Security Groups

## 檢查 docker-compose.yml 設置

我重新翻閱了官網，找到以下兩個 [靜態設置](https://doc.traefik.io/traefik/reference/static-configuration/cli/) 以及 [docker labels](https://doc.traefik.io/traefik/reference/dynamic-configuration/docker/) 這邊列出了所有的設置，我重新配置了一下我的 docker-compose.yml

```docker
version: "3.9"
services:
  app:
    image: krystallll/docker_test:1.0
    networks:
      - development
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_HOST: db
      POSTGRES_PORT: 5432
    restart: on-failure
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app-http.entrypoints=web"
      - "traefik.http.routers.app-https.entrypoints=websecure"
      - "traefik.http.routers.app-http.rule=Host(`mydocker.online`)"
      - "traefik.http.routers.app-https.rule=Host(`mydocker.online`)"
      - "traefik.http.routers.app-https.tls=true"
      - "traefik.http.routers.app-https.tls.certresolver=letsencrypt"
      - "traefik.http.middlewares.https-only.redirectscheme.scheme=https"
      - "traefik.http.routers.app-http.middlewares=https-only"
      - "traefik.http.routers.app-https.service=app"
      - "traefik.http.services.app.loadbalancer.server.port=3000"
  db:
    image: postgres:14-alpine
    networks:
      - development
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
  traefik:
    image: traefik:v2.10
    networks:
      - development
    restart: on-failure
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencrypt.acme.email=krystal@5xcampus.com"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - "./letsencrypt:/letsencrypt"
      - /var/run/docker.sock:/var/run/docker.sock
networks:
  development:
    external: true
```

以下來一一說明：

### traefik 服務的 command

1. 開起 traefik 的 Web API

```docker
--api.insecure=true
```

2. 是否暴露 docker 服務，其實翻成白話就是我不要將 docker 給 traefik 自動追蹤，只有需要被追蹤的服務再加上 `traefik.enable=true` 就好

```docker
--providers.docker.exposedbydefault=false
```

3. 建立一個名為 `web` 的入口點，他的 port 是 80

```docker
--entrypoints.web.address=:80
```

4. 建立一個名為 `websecure` 的入口點，他的 port 是 443

```docker
--entrypoints.websecure.address=:443
```

5. 由 letsencrypt 來建立憑證

```docker
--certificatesresolvers.letsencrypt=true
```

6. 使用 httpchallenge 來憑證申請

```docker
--certificatesresolvers.letsencrypt.acme.httpchallenge=true
```

7. letsencrypt 申請的憑證入口點是 web 也就是我們上面設定的 port 80

```docker
--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
```

8. 申請憑證需要的有用的 email

```docker
--certificatesresolvers.letsencrypt.acme.email=krystal@5xcampus.com
```

9. letsencrypt 申請來的憑證檔案儲存的路徑

```docker
--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
```

### app 服務的 labels

1. 上面有說到，代表這個服務歸交警伯伯 traefik 管

```docker
traefik.enable=true
```

2. `app-http` 是我取的名字，代表 `app-http` 這個東東是走 web 這條路，因為 web 正常是 http ，所以我就直接取名為 `app-http`

```docker
traefik.http.routers.app-http.entrypoints=web
```

3. 同上，代表 `app-https` 是走 websecure 這條路

```docker
traefik.http.routers.app-https.entrypoints=websecure
```

4. 設定 `app-http` 跟 `app-https` 的 Host 是 `mydocker.online`

```docker
traefik.http.routers.app-http.rule=Host(`mydocker.online`)
traefik.http.routers.app-https.rule=Host(`mydocker.online`)
```

5. 因為 `app-https` 是走 443 也就是 https 所以設定 true 代表他需要使用我們申請的憑證

```docker
traefik.http.routers.app-https.tls=true
```

6. 使用 letsencrypt 來加密解密憑證

```docker
traefik.http.routers.app-https.tls.certresolver=letsencrypt
```

7. 使用一個名為 https-only 的中介軟體，來固定將 http 轉址到 https (如果我們輸入的網址是 http 他會自動轉址到 https)

```docker
traefik.http.middlewares.https-only.redirectscheme.scheme=https
traefik.http.routers.app-http.middlewares=https-only
```

8. 將 app-https 這個 router 定義一個叫做 app 的 service

```docker
traefik.http.routers.app-https.service=app
```

9. 告訴 traefik 這個 app 的 service 是在 3000 port

```docker
traefik.http.services.app.loadbalancer.server.port=3000
```

## 重新啟動 docker-compose

```docker
docker-compose up
```

去瀏覽器輸入網址 `mydocker.online`

![搜尋 mydocker.online](/image/dockerDay27/27_3.webp)

搞了這麼多天就為了這個 https 的鎖呀 🥹🎉🎉🎉
然後再根據按鈕按一按可以得來朝思暮想的首頁。

![create database & db:migrate](/image/dockerDay27/27_4.webp)

![成功在 mydocker.online 看到專案](/image/dockerDay27/27_5.webp)

另外也可以自己試試，硬是將網址輸入 `http://mydocker.online` ，他也會自動轉址到 `https://mydocker.online` 😍😍😍
