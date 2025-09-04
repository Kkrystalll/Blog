---
title: Docker - 在 instance 上跑 Docker Image
tags:
  - Docker
  - 部署
  - Image
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
date: 2023-10-06 16:42:38
---

## 在 EC2 instance 登入 Docker Hub

```docker
docker login
```

除了可以使用之前在 [DAY 18 - 將我的 Image 推到 Docker Hub 儲存庫](https://ithelp.ithome.com.tw/articles/10333562) ，教的 `docker login` ，再根據指示輸入 username 跟 password 外，也可以用下面一句話搞定。

```docker
docker login -u < 我的 docker username > -p < 我的 docker password >
```

順利登入成功後，一樣可以看到 `Login Succeeded` 。

## 拉取我先前推上去的 image

```docker
docker pull krystallll/docker_test:1.0
```

![拉取我先前推上去的 image](/image/dockerDay21/21_1.webp)

就跟我們 git pull 大同小異，只要打對 `遠端儲存庫/image 名稱:tag` 就可以順利 pull 下 image 啦！

## 確認是否 pull 成功

```docker
docker images
```

![確認 images](/image/dockerDay21/21_2.webp)

可以看到我 EC2 instance 的本地成功有 `krystallll/docker_test`

## 迫不及待試跑看看 image

```docker
docker run -p 3000:3000 krystallll/docker_test:1.0
```

![跑看看 image](/image/dockerDay21/21_3.webp)

看起來跑成功了，但我瀏覽器要搜尋哪個網址呢？一樣是 `http://0.0.0.0:3000` 嗎？如果你試試相信會看到 404 無法連上這個網頁的字樣，因為 `http://0.0.0.0:3000` 是本機預設的，但我們現在在的地方是 EC2 inatance ，所以我們應該要去看看這個 instance 的是否有公開的 IP address 或是 DNS

![Public IP address](/image/dockerDay21/21_4.webp)

📍 之所以找 Public (公開的) IP address 或是 DNS ，而不是 Private 是因為我們希望是一個別人搜尋也可以搜尋的到的網址，所以要用 Public。

## 使用正確的網址搜尋

```
http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000
```

📍 這邊在後面加上 `:3000` 是因為我們一開始指定 port 3000

```
http://52.199.213.167:3000
```

如果有跟這我做的話，那您的這兩個應該會跟我長不一樣，如果是是複製貼上上面那段，那你只會看到我的專案喔 😜 ~~甚至我的專案內容空空如也，可能比你的還不好看 🤣~~

![確認網址](/image/dockerDay21/21_5.webp)

你不是說這是正確的網址，為什麼還是連不上 😡😡😡 其實連不上的原因可能有很多，通常是網路的問題，那因為這邊我們比較特別是指定 3000 port ，所以首先我們可以先去查這個 instance 的網路是否有允許 3000 port。

## 確認網路安全群組

### 確認 instance 的 Security 叫什麼名字

1. 勾選想確認的 instance
2. 選擇 Security Tab
3. 記住 Security groups 是哪個
   ![Security groups](/image/dockerDay21/21_6.webp)

4. 從左邊側邊欄找到 Security groups 按鈕，點進頁面
5. 找到剛剛記住的 Security groups ，並勾選
6. 點選 Actions 下拉選單
7. 選擇 Edit inbound rules 來編緝這個網路安全群組
   ![編緝網路安全群組](/image/dockerDay21/21_7.webp)

### 設置允許 3000 port 通過

1. 點選 `Add rule` 新增安全群組的規則
2. 選擇 `Custom TCP` 並在 Port range 填上 3000
3. 點選 `Save rules` 將新增的規則儲存
   ![新增安全群組規則](/image/dockerDay21/21_8.webp)

## 退出容器再重新試一次

使用之前教過的 `control 鍵 + c 鍵` 停止容器，再 `docker run -p 3000:3000 krystallll/docker_test:1.0` ，成功後在瀏覽器搜尋 `http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000`

![搜尋網頁](/image/dockerDay21/21_9.webp)

登登新的錯誤訊息來了！但是只要看到這熟悉的畫面~~(最對味)~~ ，就知道至少是 rails 的錯誤了 🥺🥺🥺
看起來是在說需要在 rails 加上設定 `config.hosts`，還貼的附上 [官方文件](https://guides.rubyonrails.org/configuring.html#actiondispatch-hostauthorization) 😍

## 設定 `config.hosts` ，重推 image

根據官方文件我們知道需要設定 `config/environments/development.rb` 或是 `config/environments/production.rb` ，那因為我們目前都是在 development ，所以我就設定 `config/environments/development.rb`

但是如果按照上面錯誤訊息那樣寫的話，本機請求的網址就會是 `ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com` 不是 `localhost:3000` ，我希望在本機時跑 `localhost:3000`，在 EC2 instance 時，跑 `ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com` ，這邊有個新學到的方法跟大家分享：

```ruby
config.hosts << "ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000" unless Rails.root.join("tmp/hostname.txt").exist?
```

他會根據 tmp 裡面有沒有 hostname.txt 這個名稱的檔案，來決定是否要呈現 `config.hosts << "ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000"` ，如果我們本機不需要 ec2 的 host ，那在 tmp 裡面新增一個名為 hostname.txt 即可。

也別忘了新增一個 `.dockerignore` 檔案，並加上：

```
/tmp/hostname.txt
```

![加上 .dockerignore](/image/dockerDay21/21_10.webp)

這樣這個檔案就不會進到 docker 裡。

📍`.dockerignore` 其實就跟`.gitignore` 是相同的概念。

完成後即可重新 build 一顆新的 image 推上 Dokcker Hub(別忘了先 docker login)

```docker
docker build -t krystallll/docker_test:1.0 . --push
```

## 在 EC2 instance 裡重新 pull 新的 image

因為剛剛上一步我們有重 push image ，所以我們需要拉最新的 image 下來運行

```docker
docker pull krystallll/docker_test:1.0
```

![pull 新的 image](/image/dockerDay21/21_11.webp)

## 重新啟動，打開瀏覽器

重複剛剛啟動容器`docker run -p 3000:3000 krystallll/docker_test:1.0` ，成功後在瀏覽器搜尋 `http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000` ，可以看到
![重新啟動，打開瀏覽器](/image/dockerDay21/21_12.webp)

沒錯這是我們熟悉的錯誤~~從沒這麼期待看到錯誤~~，那既然做到根本機一樣，接下來我們就知道，需要在 EC2 instance 裡面加上 docker compose ，這就是我們明天的內容，明天見！
