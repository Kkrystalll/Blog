---
title: Docker - 撰寫自己專案的 Dockerfile
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
date: 2023-09-27 18:58:33
---

前兩天我們了解了 Dockerfile 所有的關鍵字，就是為了今天我們實際寫的時候可以有些概念，會比較好上手一點點。
不確定大家手邊是否都有專案，所以我這邊有準備一個示範的 rails 專案，有需要的人可以從 [這邊](https://github.com/Kkrystalll/rails-docker) 將專案給 clone 下來，在預設的 main 分支就可以一步一步跟著做了。

## 新增 Dockerfile 檔案

順利打開專案後（以下範例我都是使用 Visual Studio Code 這個編輯器），首先我們需要先在專案的**根目錄**新增一個名為 **`Dockerfile`** 的檔案

![新增 Dockerfile](/image/dockerDay12/12_1.png)

📍Dockerfile 是在 Docker 中預設的 Docker 映像建置檔的預設名字，若你想要取自己想要的名字也可以，但就是在建置 image 的時候，需要說明清楚用哪個文件來建置

```docker
docker build -f MyDockerfile .
```

上面 `-f` 是 `file` 的縮寫，正常取名為 `Dockerfile` 就不用特別寫，但如果需要自定義名稱時，如： `MyDockerfile`，那就要在建置 image 時，使用 -f 標清楚

另外前幾天也有提到，我們預設是將 Dockerfile 放在根目錄，但如果我希望把 Dockerfile 放在專案路徑 docker/Dockerfile 時，就一樣需要使用 `-f` 標明清楚

```docker
docker build -f docker/Dockerfile .
```

## 確認專案版本

那我們這邊一樣根據預設在根目錄建立一個 Dockerfile 檔案就好，接著我們要來寫 `FROM` ，所以首先要先來選擇這個專案的基礎鏡像版本，首先先確認一下這個專案使用的 ruby 版本

1. 我可以在 Visual Studio Code 的終端機上使用以下語法，來查看專案的 ruby 版本

```ruby
ruby -v
```

![查看專案的 ruby 版本](/image/dockerDay12/12_2.png)

2. 若是 clone 的專案，那可以參考那個專案的 [README.md](https://github.com/Kkrystalll/rails-docker/tree/docker) ，正常都會在專案寫下使用的版本

![查看專案的 ruby 版本](/image/dockerDay12/12_3.png)

3. 或是可以在專案裡找版本或配置的檔案，在 rails 專案就可以看 Gemfile 檔案

![查看專案的 ruby 版本](/image/dockerDay12/12_4.png)

## 找尋適合的基礎鏡像版本

上面這麼多種方法我們都可以得知是使用 `ruby 3.2.2` ，那這時我們就要去 [Docker Hub ruby](https://hub.docker.com/_/ruby) 來找尋是否有符合的 image

![Docker Hub ruby](/image/dockerDay12/12_5.png)

看到後我又暈了，這麼多不同的 tag 都是 3.2.2 我到底要選用哪一個＠＠

[Docker Hub ruby](https://hub.docker.com/_/ruby) 往下滑，有介紹不同 Variants 的差別是什麼可以幫助大家選擇

![不同 Variants 的差別是什麼](/image/dockerDay12/12_6.png)

> This is the defacto image. If you are unsure about what your needs are, you probably want to use this one. It is designed to be used both as a throw away container (mount your source code and start the container to start your app), as well as the base to build other images off of.
> 根據 `ruby:<version>` 上面的說明，「如果你不確定自己的需求是什麼，你可能想使用這個」，沒錯我自己也是覺得當不知道要選擇哪個時，選擇一個最標準的 image 準沒錯，所以我在剛剛眾多的 tags 找到了我的超人！
> ![ruby:<version>](/image/dockerDay12/12_7.png)

選好之後我們可以撰寫 Dockerfile 第一行

```docker
FROM ruby:3.2.2
```

📍 特別說明上面這種版本通常是那種豪華大禮包，也就是包山包海全都有的意思，但有時候實際上我們並不需要這麼多東西，所以這時可以考慮 `ruby:<version>-alpine` ，Alpine Linux 比基礎鏡像小得多，因此可以讓鏡像更精簡，所以我更加推薦以下用法。

```docker
FROM ruby:3.2.2-alpine
```

## 寫個 LABEL 讓其他人更容易理解這個映像

```docker
LABEL maintainer="Krystal <krystal@5xcampus.com>"
```

寫個 LABEL maintainer 讓使用的人知道這個 Dockerfile 的維護者或負責人是誰（~~這個 Dockerfile 我罩的 🤣~~）
但這個部分寫不寫就見仁見智，並非必要。

## 使用 RUN 來安裝相依的軟體套件

前面兩個指令我都覺得有跡可循，或是都還在我嘗試理解的範圍內，到這邊我就覺得要開啟第三隻眼開始通靈了＠＠，真的不知道要裝些什麼誒 🫥，所以我就參考 [Docker Hub ruby](https://hub.docker.com/_/ruby) 下面寫的範例 Dockerfile

```docker
RUN bundle config --global frozen 1
```

這邊我丟了 ChatGPT 解釋一下這句話的意思，大概意思如下：

> 在這個專案中，Gemfile.lock 檔案的內容將保持不變，不會在執行 bundle install 時自動更新。

這邊我們就先將上面那句留著，先繼續寫下去（~~反正執行的時候碰到一對錯誤才會慢慢建置成對的樣子~~）

## 使用 WORKDIR 建立容器的資料夾

我就直接將資料夾取名為 app ，那未來我們進到容器內就等於直接進到這個 app 資料夾裡

```docker
WORKDIR /app
```

## 使用 COPY 來複製需要的 Gemfile

```docker
COPY Gemfile Gemfile.lock ./
```

將 Gemfile 和 Gemfile.lock 檔案複製到 Docker 映像中的目前工作目錄(app)，以便在後續的步驟中執行 bundle install 命令，安裝所需的 Gem 。
上述也等於：

```docker
COPY Gemfile Gemfile.lock .
```

也可以寫成：

```docker
COPY Gemfile* .
```

就是將所有 Gemfile 開頭的檔案，複製到當前工作目錄(app)，我自己是偏好最後一個寫法（~~因為我懶 🤣~~）

## 使用 RUN 執行 bundle install ，安裝所需的 Gem

```docker
RUN bundle install
```

之所以需要在這邊 bundle install ，是為了在 Docker 映像中的安裝 Ruby 應用程式的相依 gem 套件。主要是因為 Docker 映像是一個獨立的運作環境，所以我們要將應用程式及其依賴項放在一起備份，以確保應用在不同的環境中運行程式時具有相同的效果。

## 將專案中的所有文件 COPY 到 Docker 容器中

```docker
COPY . .
```

第一個 `.` 是**專案的目錄**，也就是我們在 Visual Studio Code 看到的那些檔案，第二個 `.` 是 Docker **容器中的當前目錄**（一起來 **想像** 有一個 app 檔案夾是當前目錄，我們要將所有東西複製進來）

## 使用 EXPOSE 告訴 Docker 監聽的 port

```docker
EXPOSE 3000
```

表示 Docker 容器在執行時應該監聽 port 3000

📍 特別注意 EXPOSE 只是一個記號，它告訴其他容器內應用程式可能使用的 port 。如果要實際將容器的 port 對應到主機上，需要在容器執行時使用 -p 或 -P 來實際執行 port 的連接。

## 用 CMD 做個收尾，設定容器啟動時要執行的預設命令

```docker
CMD ["rails", "server", "-b", "0.0.0.0"]
```

前面兩個指令就是我們一般啟動伺服器的 `rails server` ; 後面的 `-b` 是 `bind`(綁定) 的縮寫，`-b 0.0.0.0` 主要是要讓我們的應用程式可以從外部設備或其他容器可以取用。

根據上述步驟，我們大致完成了自己的 Dockerfile ，目前全貌如下：

```docker
FROM ruby:3.2.2-alpine

LABEL maintainer="Krystal <krystal@5xcampus.com>"

RUN bundle config --global frozen 1

WORKDIR /app

COPY Gemfile Gemfile.lock ./

RUN bundle install

COPY . .

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"]
```

今天我們的 Dockerfile 大概有個樣子了，但還沒結束明天要實際來根據 Dockerfile build 屬於這個專案的 image，那我們明天繼續～
