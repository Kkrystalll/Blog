---
title: Docker - 將 Dockerfile 改成多階段建置
tags:
  - Docker
  - 部署
  - Dockerfile
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-14 21:58:37
---

## 多階段建置的優點

### 減少鏡像大小：

多階段建置可以在一開始建置階段使用標準版本的鏡像，讓建置階段時，就可以下載好需要的工具和依賴套件，然後在第二階段時，就可以換到輕量級的 Alpine 鏡像。
如此一來可以將應用程式，以及只包含執行時所需的依賴套件，可以不包括多餘的檔案，可以減小鏡像的大小。

### 提高安全性：

當我們可以減少容器的鏡像大小，就代表著減少了潛在的風險，所以可以提高安全性。

### 最佳化建置速度：

這個是我覺得多階段建置的最大優點，便是當檔案沒有變動時，在 build 時就不用重複 build ，所以可以減省建置的時間，進而提高速度。

### 更清晰的建置流程：

將建置流程劃分為多個階段，可以使 Dockerfile 更加清晰且易於維護。

總而言之，多階段建置是一種最佳化 Docker 鏡像建置的方法，可以減少鏡像大小、提高安全性、優化建置速度並提供更流暢的建置流程，可以減少資源消耗並加快速度部署速度。

## 實際修改 Dockerfile

原本沒有使用多階段建置的 Dockerfile

```docker
FROM ruby:3.2.2-alpine

LABEL maintainer="Krystal <krystal@5xcampus.com>"

RUN apk add \
    build-base \
    postgresql-dev \
    tzdata

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN bundle install

COPY . .

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

修改後的 Dockerfile：

```docker
# 第一階段：建置階段
FROM ruby:3.2.2 as builder

RUN apk add --no-cache \
    build-base \
    postgresql-dev \
    tzdata

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN bundle install

COPY . .

#階段2：運行階段
FROM alpine:3.14

RUN apk add --no-cache tzdata

# 從建置階段複製編譯好的應用程式
COPY --from=builder /app /app

WORKDIR /app

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]

```

這邊幾個不同之處說明一下

```docker
FROM ruby:3.2.2 as builder
```

建置階段可以使用標準版本的 Ruby 鏡像進行構建，並且叫做 builder

```docker
RUN apk add --no-cache \
    build-base \
    postgresql-dev \
    tzdata
```

`--no-cache` 是安裝時，不緩存下載的索引檔案，所以可以有效減小鏡像大小

```docker
FROM alpine:3.14
```

使用輕量的 Alpine 鏡像作為最終鏡像

```docker
COPY --from=builder /app /app
```

將前一個建置階段（builder）中的檔案或目錄複製到目前建置階段

接著就可以重新 build image 重新推上 Docker Hub ，在 EC2 重新拉下來啟動容器 ; 或是在本地直接 `docker-compose up`。

那多階段就介紹到這邊，明天見～～
