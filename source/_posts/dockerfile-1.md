---
title: Docker - 拆解 Dockerfile 的關鍵字(上)
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
date: 2023-09-25 18:16:03
---

## Dockerfile 是什麼？

回到我前幾天介紹的 Docker Image(映像) 我們把它想像成烤雞蛋糕模型，那這個 Dockerfile 我們就可以把它想像成這個模型的說明書，這本說明書包含了一系列指令，告訴 Docker 如何建立一个映像。當我們要 run docker 時，他會根據現在專案的根目錄，找尋 Dockerfile ，並且根據 Dockerfile 裡面一步一步的說明，來安裝好相關套件資料庫等環境，然後製作成一個 Image(映像)。
當然如果你的 Dockerfile 沒有在根目錄也是可以的，只要指定對路徑，能找到 Dockerfile 就可以根據 Dockerfile 來製作 Image(映像)。

## Dockerfile 裡面的關鍵字

### FROM

也就是我們在 Docker Hub 上可以看到的基礎鏡像版本

```docker
FROM ruby:3.2.2
```

📍 通常會放在 Dockerfile 的第一行

### AS

通常與 FROM 一起在多階段建立，我們可以把它想像成暱稱

```docker
FROM ruby:3.2.2 AS app
```

也就是我們可以在這個檔案後面以 `app` 這個暱稱來稱呼 `ruby:3.2.2`

### MAINTAINER(目前建議使用 LABEL)

Dockerfile 的作者/維護者（1.13 版本以後，建議改用 LABEL）

```docker
MAINTAINER Krystal <krystal@example.com>
```

### LABEL

以 key value 的形式，來描述鏡像的屬性(作者、版本、描述等)

```docker
LABEL maintainer="Krystal <krystal@example.com>"
LABEL version="1.0"
LABEL description="My rails project"
LABEL website="https://www.example.com"
```

主要功用就是提供資訊，讓其他人更容易理解和使用映像，我自己是把他想得很像 README 的作用。

### RUN

建立 image 時，安裝軟體套件、下載依賴項、設定環境。

```docker
RUN apk add --update --no-cache \
    postgresql-dev

RUN gem install bundler:2.3.19 && \
    bundle install -j4 --retry 3 && \
    bundle clean --force
```

📍 每個 RUN 指令都會 **生成一層新的鏡像層**，所以根據上面範例會生成兩層

📍 上面看到的 `\` 其實只是為了排版美觀的一個符號，代表雖然換行，但其實是同一行。
所以我們可以改成如下

```docker
RUN apk add --update --no-cache postgresql-dev

RUN gem install bundler:2.3.19 &&  bundle install -j4 --retry 3 && bundle clean --force
```

只是我個人推薦上方有 `\` 符號的寫法，因為可讀性較佳。

📍 `&&` 其實就是如果前一個命令執行成功，才會執行下一個命令。如果前一個命令執行失敗，那後面的命令就不會執行。

上面會需要 && 是為了確保安裝 bundler、執行 bundle install 和清理 bundle 時，只有在前一步成功時才繼續執行後續步驟。

### CMD

容器啟動時要執行的預設命令，每個 Dockerfile 只能有一個 CMD 指令，如果有多個 CMD 指令，前面會被覆蓋，只有最後一個會執行

```docker
CMD ["rails", "server", "-b", "0.0.0.0"]
```

也因為上面說的 CMD 指令會被覆蓋，所以我們通常把他放在 **Dockerfile 的最後一行**，以避免覆蓋狀況。

### ENTRYPOINT

定義容器的主要執行命令，且會在容器啟動時執行

```docker
ENTRYPOINT ["echo", "Hello, Docker!"]
```

當 Docker run 這個容器，容器啟動後便會輸出 "Hello, Docker!"

看到這邊我超暈 CMD 跟 ENTRYPOINT 差在哪？看起來根本都一樣，我們直接用範例解釋

#### CMD 跟 ENTRYPOINT 的差別

如果有上面的 ENTRYPOINT ，然後我們將這個 image run 起來 `docker run <image> Hi`，再多給一個 Hi 參數
會輸出

```docker
Hello, Docker! Hi
```

反過來如果是 CMD

```docker
CMD ["echo", "Hello"]
```

然後我們將這個 image run 起來 `docker run <image> Hi`，再多給一個 Hi 參數

會輸出

```docker
Hello Hi
```

你可能覺得看起來都一樣，但其實不同的是，這時 CMD 的內容 `["echo", "Hello"]` 已經被修改為 `["echo", "Hello", "Hi"]`

總結來說 ENTRYPOINT 是不可變且一定會執行的 ; CMD 可能被覆蓋也可能被修改的。

### EXPOSE

指出容器內應用程式，應該使用哪些 port (連接口)進行監聽

```docker
EXPOSE 3000
```

代表監聽 3000 port

今天就先介紹到這邊，明天我們繼續拆解 Dockerfile 剩餘的關鍵字～
