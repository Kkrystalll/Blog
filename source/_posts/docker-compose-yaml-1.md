---
title: Docker - 撰寫 Docker Compose 的 YAML 檔案(上)
tags:
  - Docker
  - 部署
  - docker-compose
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
date: 2023-10-01 23:53:14
---

## 新增 Docker Compose YAML 檔案

一直說 YAML 檔案，但 YAML 到底是什麼？

📍 YAML 輕量且易於撰寫的檔案，主要可以將資料轉換成特定格式，常用於設定檔和配置檔的編寫。

跟 Dockerfile 一樣，我們可以新增一個 `docker-compose.yml` 或 `docker-compose.yaml` 檔案在根目錄，原因不外乎就是因為 `.yml` 或是 `.yaml` 都可以被識別為是 YAML 檔，那我秉持一貫懶惰的個性，就推薦直接用 `.yml` 為結尾。

![新增 Docker Compose YAML](/image/dockerDay16/16_1.png)

## 確定 docker-compose.yml 版本

```docker
version: "3.9"
```

表示正在使用 Docker Compose 設定檔版本為 3.9。主要是因為不同的版本可能支援不同的功能和語法，所以確保版本是非常重要的。

### 要如何知道我的 docker-compose 版本？

```docker
docker-compose -v
```

或

```docker
docker-compose --version
```

![我的 docker-compose 版本](/image/dockerDay16/16_2.png)

### 為何我的版本是 2.10.2 上面寫 3.9？

Docker Compose 配置文件的版本 (version) 與 Docker Compose 工具的版本 (docker-compose -v) 並不一定要完全一樣。
版本號是用來指示配置文件支援的語法和功能的，這表示配置文件使用了 Docker Compose 版本 3.9 定義的語法和功能。

📍 通常情況下，較新版本的 Docker Compose 工具可以處理較舊版本的配置文件，因為它們通常是相容的，所以這邊選擇較新版本 3.9 是 ok 的。

## 定義不同容器的 services (服務)

services 是用來定義不同容器服務的部分。容器內會包括容器運行的相關配置、映像、環境變數、port、依賴關係等資訊。

舉例來說就是，以我們準備的這個專案，services 底下就會有兩個：

### 1. 是 rails 的容器

也就是我們那天用 Dockerfile 建立的 image ，我們可以透過 image 來 run 成容器

### 2. 是 PostgreSQL 的容器

之所以前天最後留在紅畫面的原因正是如此，因為我們並沒有 PostgreSQL 的容器，又或者是 PostgreSQL 的容器，並沒有好好的跟 rails 容器連線，那沒有連上線其實就等於沒有 PostgreSQL 的容器。

所以我們接著就要來定義兩個容器，並讓兩者順利連線！

## 先將 rails 的容器定義好

### 定義服務名稱

在 services 底下的每一個 service 都會先定義一個在這個 Compose 檔案中唯一的名稱，這個名稱是為了區分不同的容器化服務。

```docker
app:
```

這邊我就將我的專案服務取名為「app」。（當然這個想叫什麼名字都可以由大家自行命名）

### 根據什麼建立容器？

我們在之前已經寫好 Dockerfile ，所以我們可以根據 Dockerfile 製成的 image ，建立成容器

```docker
build:
  context: .
```

上面分開解釋就是
**build** ：如何建立容器？
**context** ：根據什麼建立容器？
**.** ： 當前目錄

就可以寫成如上， Compose 文件將根據當前目錄中尋找名為 Dockerfile 的文件，然後使用這個 Dockerfile 來建立容器。

### 設置 compose 檔的環境變數

### 先設定專案的 database.yml 檔

`config/database.yml` 是做什麼呢？
這個檔案的是告訴 Rails 應用程式應該如何連接到資料庫、使用哪種資料庫管理系統、資料庫的名稱、使用者名稱、使用者密碼等等，所有跟資料庫連接的參數們。

原本的 `config/database.yml` 如下

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: docker_test_development

test:
  <<: *default
  database: docker_test_test

production:
  <<: *default
  database: docker_test_production
  username: docker_test
  password: <%= ENV["DOCKER_TEST_DATABASE_PASSWORD"] %>
```

我們需要將連接相關資訊寫成環境變數 ENV (機密資料寫成環境變數較安全)，會改成如下：

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV['POSTGRES_USER'] %>
  password: <%= ENV['POSTGRES_PASSWORD'] %>
  host: <%= ENV['POSTGRES_HOST'] %>
  port: <%= ENV['POSTGRES_PORT'] %>

development:
  <<: *default
  database: docker_test_development

test:
  <<: *default
  database: docker_test_test

production:
  <<: *default
  database: docker_test_production
  username: docker_test
  password: <%= ENV["DOCKER_TEST_DATABASE_PASSWORD"] %>
```

我們根據 `<%= ENV['我設定的環境變數 key'] %>` 可以取用到當個環境變數的 value，但是內容會在哪呢？需要我們自己定

在專案根目錄下新增一個 `.env` 的檔案，並把環境變數寫進去

```yml
# database settings
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_HOST=db
POSTGRES_PORT=5432
```

`POSTGRES_USER`、`POSTGRES_PASSWORD` 無論是環境變數 key 或是 value 都可以跟我不一樣，也就是可以自己取

`POSTGRES_HOST` 通常本地會是 `db` 或是 `localhost` 但沒有一定

`POSTGRES_PORT` PostgreSQL 預設的 port 就是 5432

📍 特別提醒 📍
若有環境變數的內容建議將 .env 檔加到 `.gitignore` 裡，避免你的機密資訊加進 Git 版控

以上都完成我們可以將環境變數加到 compose 裡

```docker
environment:
   POSTGRES_USER: postgres
   POSTGRES_PASSWORD: password
   POSTGRES_HOST: db
```

當然可以直接都寫出來，但 compose 檔也支援環境變數，所以也可以換成環境變數

```docker
environment:
   POSTGRES_USER: ${POSTGRES_USER}
   POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
   POSTGRES_HOST: ${POSTGRES_HOST}
```

目前 compose 檔撰寫成如下：

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
```

那今天就先介紹到 app 服務的撰寫，明天繼續將資料庫的服務也一起撰寫完，並試著使用 docker compose 將多個容器同時啟動，看能否順利打開專案！
