---
title: Docker - 將我的 Image 推到 Docker Hub 儲存庫
tags:
  - Docker
  - 部署
  - Docker Hub
  - docker private registry
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-10-03 18:20:33
---

為了要做到可以部署，我們需要先將我們專案包好的 image 推到 Docker Hub 的私有儲存庫，供之後可以來這邊拿取並啟動容器。翻成白話大概就是我要先把我的東西先放在倉庫，這樣之後只要有倉庫鑰匙的人，都可以去倉庫取貨。

## 註冊/登入 Docker Hub

搜尋 [Docker Hub](https://hub.docker.com/) ，可以點選右上角的註冊或登入

![註冊/登入 Docker Hub](/image/dockerDay18/18_1.png)

若是跟我一樣已經有登入過，記住帳號的人，我們可以看到圖片中黃色框，他很貼心的顯示卡片可以讓我快速登入

## 建立私有儲存庫

登入後便會看到我所有的儲存庫，如下圖可以看到有兩條白白的，代表我有兩個儲存庫了，但因為有些資料不方便透露，所以我這邊先填白色，正常來說會有儲存庫的名稱等相關資料。

![建立私有儲存庫](/image/dockerDay18/18_2.png)

無論你有沒有儲存庫都沒關係，我們可以直接點選右上方 `Create repository` 建立自己的儲存庫

![建立自己的儲存庫](/image/dockerDay18/18_3.png)

根據圖片的步驟：

1. 填入儲存庫的名稱
2. 選擇儲存庫是公開還是私有
   📍 選擇公開就代表其他人在 Docker Hub 是可以直接搜尋到你的 Image 喔
   📍 若是免費方案，一個帳號只能建一個私有儲存庫，如果希望可以有更多儲存庫可以參考 [付費方案](https://hub.docker.com/billing/plan/update)

![付費方案](/image/dockerDay18/18_4.png)

一個月 5 美元，就可以建無上限的私有儲存庫 3. 點選 `create` 建立

建立後會直接進到這個儲存庫內，可以看到這個儲存庫的詳細資訊。

![儲存庫的詳細資訊](/image/dockerDay18/18_5.png)

回到我的儲存庫列表，也可以看到我剛剛建的 `docker_test`

![儲存庫列表](/image/dockerDay18/18_6.png)

## 將我的 image 推到 Docker Hub 儲存庫

我們可以先想像一下 Git Hub ，當我每次要將分支推到 Git Hub 上時，總是會需要輸入密碼，也就是要先身份驗證 ; 那在 Docker Hub 上就是反過來，在推送之前就要先進行身份驗證，也就是

```docker
docker login
```

當打了 `docker login` 會依序要您輸入 `Username` 跟 `Password`

![docker login](/image/dockerDay18/18_7.png)

登入成功即會看到如圖的 `Login Succeeded`

![Login Succeeded](/image/dockerDay18/18_8.png)

看到前面建立的儲存庫有給提示，可以直接使用 `docker push krystallll/docker_test:tagname` 但是我們還沒有 krystallll/docker_test 這個 image，所以我們可以使用兩種方法

1. 直接 build 一個 krystallll/docker_test 的 image

```docker
docker build -t krystallll/docker_test:1.0 .
```

當 build 好時，看到了

> WARNING: No output specified with docker-container driver. Build result will only remain in the build cache. To push result image into registry use --push or to load image into docker use --load

其實這跟我們之前寫 Dockerfile 時遇到的問題一樣，但現在我們是要將他推上 Docker Hub ，所以這次改使用 `--push`

```docker
docker build -t krystallll/docker_test:1.0 . --push
```

![docker build](/image/dockerDay18/18_9.png)

2. 將 build 好的 image 使用 docker tag 給一個新標籤

```docker
docker tag my-ruby:1.0 krystallll/docker_test:1.0
```

如果這句看不太懂可以複習 [Day 7 - Docker Tags(標籤) 不只是標籤](https://ithelp.ithome.com.tw/articles/10324739)🧐

📍 需要特別注意本來的 my-ruby:1.0 是從最新的 Dockerfile 建成的

以上兩種方法二擇一就好，都可以達成一樣的效果，我個人是選第一種，因為我金魚腦，忘記 my-ruby:1.0 是否是從最新的 Dockerfile build 成的，所以乾脆直接選擇前者。

完成後我們可以去 docker_test 得 repository 看，可以看到剛剛有順利推送上來了

![docker_test 得 repository](/image/dockerDay18/18_10.png)

今天順利將 image 推上 Docker Hub ，明天就要來認識 Amazon Elastic Compute Cloud（Amazon EC2）的服務，來一步步實現部署大業！
