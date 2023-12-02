---
title: Docker - 拆解 Dockerfile 的關鍵字(下)
tags:
  - Docker
  - 部署
  - Dockerfile
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-26 20:08:40
---

昨天了解上半部的 Dockerfile 關鍵字，今天繼續來把 Dockerfile 的關鍵字學完吧～

### ENV

在容器內部設定的環境變數

```docker
ENV DB_HOST=postgresql
```

設定一個環境變數叫做 `DB_HOST` ， value 是 `postgresql`

### ARG

在 Dockerfile 中使用的參數，與 ENV 作用很類似，但是作用域不一樣。
ARG 設定的環境變數只有在 Dockerfile 內，及 docker build 的過程中有效，建立成 Image(映像) 內，就沒有這個環境變數 ; 但 ENV 則是容器啟動時，也可以使用的變數。

```docker
ARG <参数名>[=<默认值>]
```

```docker
ARG DB_HOST=postgresql
```

### WORKDIR

是設定這個容器內部現在的資料夾

```docker
WORKDIR /app
```

代表 Docker 會在容器裡面建立一個名為 /app 的資料夾，且現在就在這個資料夾裡，我自己是把它想像成這個容器的根目錄。

### COPY

將文件或文件夾複製到 Image(鏡像) 裡

```docker
COPY app.rb /app
```

將 app.rb 檔案複製到 app 資料夾裡

### ADD

也是將文件或文件夾複製到 Image(鏡像) 裡，除了複製外，還支援自動解壓縮檔案、URL 下載和複製上下文的檔案

```docker
ADD https://example.com/file.gz /app/
```

自動解壓縮 `https://example.com/file.gz` 這個檔案，並下載下來到 app 目錄內

### VOLUME

可以建立一個或多個永久性儲存的資料夾，內容不會隨容器的生命週期結束而消失，裡面的資料可以在容器與容器間共享。
📍 但 VOLUME 裡面的資料並不是存在容器裡，是存在主機的文件裡。

```docker
VOLUME /mydata
```

建立一個 `mydata` 資料夾，可以把需要一直存在的資料放進來。

如果希望容器中的資料在容器停止後仍然存在，就使用 VOLUME 來建立永久性存儲區 ; 而 WORKDIR 只有用於操作容器內部的當前工作目錄，會隨容器的生命週期結束而消失。

### USER

指定用哪個使用者身分來執行容器中的命令

```docker
USER <用户名稱或用户 ID>:<群組名稱或群组 ID>
```

```docker
USER krystal:baby_team
```

也可以單純指定 user

```docker
USER krystal
```

優點是可以限制容器內部使用者，根據身分來有不同權限執行，可以增加容器的安全性。

### ONBUILD

> 在基礎映像建置時要執行的操作，讓您在建置基礎映像時當時定義了一些操作，然後在後續的鏡像建置中自動執行這些操作。

上面這段解釋老實說我是完全有看沒有懂，直接使用下面範例來說明:

我們先將這個生成的映像取名為 `my-base-image`

```docker
FROM ubuntu:latest

ONBUILD ADD install_app /app/
ONBUILD RUN /app/install_app
```

ONBUILD 兩句依序的意思是：

1. 將原本映像的 install_app 複製到新映像的 /app/ 目錄中。
2. 執行新映像中的 /app/install_app 。

但重點是他何時執行？那就是當我要建立另一個新的映像，而這個映像是取自上一個建立的 `my-base-image` 映像時，我的 Dockerfile 可以直接這樣寫

```docker
FROM my-base-image
```

這時我的新映像便會直接執行前面 ONBUILD 的那兩句指令，也就是我不用在這邊在寫一次。
優點是可以 **通用邏輯** ; 缺點是會更複雜，導致有非預期的錯誤發生。

### STOPSIGNAL

如其名，是容器在接收到停止時，需要做的動作

```docker
STOPSIGNAL SIGTERM
```

預設是 SIGTERM ，就是單純的停止。

```docker
STOPSIGNAL 9
```

9 就是 立即強制停止。

### HEALTHCHECK

可以在容器內部執行一些自訂的指令或檢查，來檢查容器的健康狀態。

```docker
HEALTHCHECK --interval=30s --timeout=10s --start-period=90s --retries=5 CMD [ "ruby", "health_check.rb" ]
```

#### --interval=30s

這表示健康檢查的 **時間間隔** 為 30 秒，也就是每 30 秒健康檢查一次。
📍 預設為 30 秒

#### --timeout=10s

如果健康檢查時間超過 10 秒，則為失敗。
📍 預設為 30 秒

#### --start-period=90s

通常容器啟動會需要一些時間，但如果我們設定每 30 秒檢查一次，那可能容器都還沒有順利啟動好，就檢查了，就可能造成不正確的檢查結果，所以我們可以透過 `--start-period` 來設定容器啟動後，健康檢查之前的等待時間。
所以上面的意思就是容器啟動後再等 90 秒，再開始健康檢查。
📍 預設為 0 秒

#### --start-interval=10s

在容器啟動後的多長時間第一次進行健康檢查。
📍 預設為 5 秒

#### --retries=5

表示若是健康檢查為不健康時，會再重試幾次。
如範例就是如果在 5 次連續的健康檢查中都失敗了，容器將被標記為不健康。
📍 預設為 3 次

接著會實際執行 `ruby health_check.rb` 這個檔案的指令來做健康檢查，如果成功，則容器標示為健康。

### SHELL

在 Dockerfile 中執行命令時使用的預設 shell，通常是用在 Windows 上要基於 Linux 的容器映像時，可以將預設 shell 變為 Bash。

```docker
SHELL ["/bin/bash", "-c"]
```

光是要一一介紹 Dockerfile 的關鍵字，就花了兩天的篇幅呢！但我想若是可以先詳細介紹完這些關鍵字的功用，當我們在撰寫 Dockerfile 時，也比較知道應該怎麼寫，以及我到底在寫什麼，所以明天我們就要來撰寫 Dockerfile 啦，明天見～～～
