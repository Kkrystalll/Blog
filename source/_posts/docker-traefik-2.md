---
title: Docker - 在我部署的專案使用 Traefik 取得 HTTPS 協定(二)
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
date: 2023-10-10 12:30:26
---

昨天我們成功在
`http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata` 成功看到 API 原始資料，那今天首先就先去 `http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000` 看看

![Traefik 配置](/image/dockerDay22/22_9.webp)

因為昨天有 --build 的關係，所以會跑出這個需要我們建 db 是預料中的。到這邊為止目前看起來都沒有異常，但我們一開始使用 traefik 是因為我們需要將網址從 http 變成 https ，所以我們將網址改為 `https://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000` 再試一次

![Traefik 配置](/image/dockerDay25/25_2.webp)

去看看 log

![Traefik 配置](/image/dockerDay25/25_3.webp)

> HTTP parse error, malformed request: #<Puma::HttpParserError: Invalid HTTP format, parsing fails. Are you trying to open an SSL connection to a non-SSL Puma?>

可以猜測這應該是一個無效的 http 請求，為甚麼會無效呢？我重新看一下 [官網](https://doc.traefik.io/traefik/getting-started/quick-start/) 的圖片

![Traefik 配置](/image/dockerDay25/25_4.webp)

可以看到交警伯伯 traefik ，當我們訪問網址時，會找到交警伯伯，但是我們好像還沒有將我們這個網址的服務 app 跟他做連結，所以我需要先將我的 app 服務與 traefik 做個連結

## 將 app 服務跟 traefik 連結

我找到一篇蒼時弦也的 [Rails 部署實踐 - 使用 HTTPS 協定加密連線](https://blog.aotoki.me/posts/2022/03/25/rails-deployment-in-practice-enable-https/) 與我要達成的目標相近，所以參考這篇文章並將其改為符合我專案的樣子，目前 docker-compose.yml 長成如下：

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
      # ports:
      # - 80:3000
    labels:
    - "traefik.enable=true"
    - "traefik.http.services.app.loadbalancer.server.port=3000"
    - "traefik.http.routers.app.rule=Host(`ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com`)"
    - "traefik.http.routers.app.entrypoints=web"
    - "traefik.http.routers.app.tls=true"
    - "traefik.http.routers.app.tls.certresolver=letsencrypt"
  db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
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
```

這邊將原本 middlewares 有關的都先拿掉，一方面是因為我在下方的錯誤訊息裡撞牆了好久

> ec2-user-traefik-1 | time="2023-10-10T17:54:19Z" level=error msg="middleware \"https-redirect@docker\" does not exist" entryPointName=web routerName=app-http@docker

另外也因為，希望是先用最簡化的方式教大家部署，一種先求有再求好的心態，所以我這邊就先移除 middlewares。

當我重新 `docker-compose up` 時，終於脫離上述的錯誤泥沼中，看到新的錯誤

> ec2-user-traefik-1 | time="2023-10-10T18:40:38Z" level=error msg="Unable to obtain ACME certificate for domains \"ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com\": unable to generate a certificate for the domains [ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com]: acme: error: 400 :: POST :: https://acme-v02.api.letsencrypt.org/acme/new-order :: urn:ietf:params:acme:error:rejectedIdentifier :: Error creating new order :: Cannot issue for \"ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com\": The ACME server refuses to issue a certificate for this domain name, because it is forbidden by policy" ACME CA="https://acme-v02.api.letsencrypt.org/directory" routerName=app@docker rule="Host(`ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com`)" providerName=letsencrypt.acme

看起來像是他無法發給我證書的錯誤訊息，那我們明天繼續來找找為何會無法發放吧！
