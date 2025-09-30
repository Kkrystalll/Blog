---
title: Docker - 要取得協定得先有正確的 Domain
tags:
  - Docker
  - 部署
  - Domain
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
  - Domain
date: 2023-10-11 18:27:17
---

昨天結尾的錯誤訊息説，無法發給 `ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com` 這個 domain 正確的 ACME 證書。

所以推測可能我們使用的 AWS DNS 並沒有符合 Let's Encrypt ，所以我們會需要一個正常的 Domain ，那我這邊是直接在 [Godaddy](https://tw.godaddy.com/) 選購，我買了一個 `mydocker.online
` 的 Domain

![Godaddy Domain](/image/dockerDay26/26_1.webp)

購買完成後，還需要在 Godaddy 設定 AWS Public IP address

## 在 Godaddy 管理 DNS

以下步驟我是參考從谷歌大神那邊問來的 [文章](https://saturncloud.io/blog/connect-your-aws-ec2-instance-to-a-domain-on-godaddy-a-comprehensive-guide/)

### 1. 點選右上角頭像到我的產品

### 2. 點選這個 domain 的 DNS 按鈕

![domain DNS](/image/dockerDay26/26_2.webp)

### 3. 點選鉛筆編輯

### 4. 類型選 `Ａ`

### 5. 名稱 `@`

### 6. 資料就是填入我們在 EC2 的 Public IP address

### 7. TTL 選擇自訂 600 秒（最短就是 600 秒）

![Godaddy 管理 DNS](/image/dockerDay26/26_3.webp)

## 修改 hostname 重新 build image

等待的同時我們可以先去本來的專案，將 `development.rb` 之前設定的

```ruby
config.hosts << "ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000" unless Rails.root.join("tmp/hostname.txt").exist?
```

改為

```ruby
config.hosts << "mydocker.online" unless Rails.root.join("tmp/hostname.txt").exist?
```

因為我們現在有換新的 domain 所以這個才需要修改，然後重新 build image 推上 Docker Hub (記得先 login)

```docker
docker build -t krystallll/docker_test:1.0 . --push
```

## pull 新的 image 到 EC2 inatance 上

在 EC2 inatance 裡，先 login 到 Docker Hub，再將最新的 image 拉下來

```docker
docker pull krystallll/docker_test:1.0
```

## 修改 EC2 inatance 的 docker-compose.yml

將本來的

```docker
- "traefik.http.routers.app.rule=Host(`ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com`)"
```

改為

```docker
- "traefik.http.routers.app.rule=Host(`mydocker.online`)"`
```

並且將 app 的 ports 改為

```docker
ports:
    - 80:3000
```

在 docker 的 80 port 對到 app 的 3000 port，並先把 `traefik` 服務都先註解掉，我先確認這個 domain 確定可以連接，這時使用

```docker
docker-compose up
```

![確認 domain 可以連接](/image/dockerDay26/26_4.webp)

📍 我們在設定 Godaddy 選擇了 600 秒，代表 600 秒後才會生效，所以若沒畫面也可能是還沒有成功生效，他最晚一天之內一定會好，如果沒好那可能就是你沒設定好 🥺

既然 Domain 都用好了，明天又要重回 traefik debug 地獄 🤮🤮🤮
