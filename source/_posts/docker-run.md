---
title: Docker - 將我的 image 跑起來
tags:
  - Docker
  - 部署
  - Dockerfile
  - docker build
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-29 21:44:36
---

將建立好的 image 製成 container
前面使用 `docker build -t my-ruby:1.0 . --load` 只是單純建立好模型，但是當你希望他實際上可以運轉時，還是要使用 `docker run` 的指令將 image 啟動一個新的 container

```docker
docker run my-ruby:1.0
```

得到以下輸出

> tzinfo-data is not present. Please add gem 'tzinfo-data' to your Gemfile and run bundle install (TZInfo::DataSourceNotFound)
> 現在已經對錯誤麻木了，我把錯誤訊息貼給谷歌大神，再加上 docker 這個關鍵字，[第一個搜尋結果](https://gist.github.com/skozz/0ac405c565b41bfa21fb93e32ab69ad4)答案就出來了！
> 原來我前面因為不認識所拿掉的 `tzdata` 是負責處理時區的套件，而這也是相依套件，所以我們再把它加回 Dockerfile ，重新 buile 跟 run 試試看

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

雖然這個過程可能很繁瑣，但是我們在建置 Dockerfile 時，真的實際就是這樣，一開始不知道有哪些相異套件要用 RUN 裝進容器裡，都是一次次的錯誤，一個個加上去的。

![docker run](/image/dockerDay14/14_1.png)
![docker run](/image/dockerDay14/14_2.png)

根據截圖看起來幾乎是成功了！他甚至跟我說 `Listening on http://0.0.0.0:3000` 我還不手刀打開瀏覽器，輸入網址 `http://0.0.0.0:3000`

![docker run](/image/dockerDay14/14_3.png)

關鍵字搜尋 again `docker run image localhost not found`，看到這篇 [stackoverflow](https://stackoverflow.com/questions/69341721/running-a-docker-container-on-localhost-not-working) ，決定試試

```docker
# docker run -p <host-port>:<container-port> imageName
docker run -p 3000:3000 my-ruby:1.0
```

因為 rails 是 3000 port，所以我們要提醒他要監聽的 port

## 若要使用相同的 port ，要先停掉本來的 container

因為我們要重新 run 一個相同的 port，又或是說現在這個 container 我們不用了，所以要先停止或退出容器，可以使用以下方法退出

1. Ctrl-C
2. docker stop <container_id 或 container_name>

順利退出後會有 `- Goodbye! Exiting` 的字樣，這時再重新使用 `docker run -p 3000:3000 my-ruby:1.0` 重新開起新的容器，並告訴他要監聽的 port 是 3000 port，再次順利開起，打開瀏覽器，輸入網址 `http://0.0.0.0:3000`

![docker run](/image/dockerDay14/14_4.png)

至少，有錯誤訊息，而且這個錯誤看起來像是 rails 專案會看到的紅畫面，裡面說的內容感覺跟資料庫有關，算是一個大進步的錯誤訊息，至少代表我們成功用 docker 開啟 rails 專案了！！！至於怎麼處理這個錯誤，明天再繼續說明。
