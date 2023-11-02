---
title: Docker - 管理 Docker Container(容器) 的生命周期
tags:
  - Docker
  - 部署
  - container
comments: false
toc: true
cover: /image/docker.png
categories:
  - Docker
  - 部署
date: 2023-09-20 00:31:03
---

## Docker Container 的啟動、開始、停止、刪除

### 啟動 Container

當我們想要執行一個 Container 容器時，會使用 `docker run` 這個指令，這個指令會從我本地的 image 或 Docker Hub 儲存庫中下載名為 ruby 的 image ，然後使用這個 image 建立一個新的 container 並執行它。

```
docker run ruby
```

以上面的指令為例，會先看我本地是否有名為 `ruby` 的 image ，若有就根據這個 image 生成 container 並執行它 ; 若無，就去 Docker Hub 儲存庫中下載名為 ruby 的 image ，然後建立新的 container 並執行它。
![Docker run](/image/dockerDay5/5_1.png)

根據上面截圖的第一句可以看到 `Unable to find image 'ruby:latest' locally` 代表本地沒有找到 `ruby:latest` 的 image ，所以可以看到第二句 `latest: Pulling from library/ruby` ，代表從 Docker Hub 的 library/ruby 存儲庫中下載 `ruby:latest` 映像。
![Docker run 自動下載 latest](/image/dockerDay5/5_2.png)

然後執行完成我可以可以看到上面那些亂數，也就是 image 內的每個層（layers）都 `Pull complete` 拉取完成了，代表我們已經成功在運行 ruby 這個 container 了。

到這邊不知道大家會不會跟我一樣好奇，我剛剛明明就是下 `ruby` 這個 image ，為何上面的 log 卻是去找 `ruby:latest` 這個 image ? `:latest` 究竟是什麼？哪來的？

#### docker image tag

其實這個 `:latest` 就是 image 的 **tag(標籤)**，我們可以把我們的 image 取名為不同的 tag(標籤)，正常來說這個 tag 會是版本號，而當我們都不特別下 tag 時，則會去找預設的 tag ，也就是 `:latest` ，這就是為什麼我們上面沒有特別下 tag 他會自己去找 `ruby:latest` 的原因。

```
docker run ruby:3.2.2
```

我們可以在自己的儲存庫或是 Docker Hub 看到有哪些 tag 可以用，如：[搜尋 Docker Hub ruby](https://hub.docker.com/_/ruby)，可以看到 Supported tags。
以上面範例程式碼為例，就是執行 ruby image 3.2.2 版本的 tag。

那既然我前面有成功 `docker run ruby` ，我預期我現在使用 `docker ps` 來查看我目前運行中的 container ，會看到我剛剛跑成功的 ruby ：
![看我目前運行中的 container](/image/dockerDay5/5_3.png)

結果怎麼是空的？？？我到底有沒有成功執行啊？
我們先使用 `docker ps -a` 指令來看一下所有 container 的狀態
![看一下所有 container 的狀態](/image/dockerDay5/5_4.png)

這邊有看到我的 ruby image ，但是看看 status 是 `Exited(已退出)` ，這是因為 container 會在執行完任務後就退出，以 ruby 為例，就是在執行完 irb（交互式 Ruby 解釋器）後會自動退出，所以若想要讓他不自動退出，就需要使用交互模式(可以進到容器內部互動)執行，或是有個讓他可以持久運行的東西。
翻成白話大概就是，如果沒事做 container 就下班，如果有事做 container 就會做完再下班。

#### 交互模式(可以進到容器內部互動)執行

```
docker run -i -t ruby /bin/bash
```

上面的指令也可以省略成如下

```
docker run -it ruby /bin/bash
```

**-i**：代表 "交互式"（Interactive）。這個指令可以讓你輸入指令。如果沒有使用 -i ，則容器不會接受輸入的指令（就是你怎麼輸入他都不理你）。

**-t**：代表 "終端"（Terminal）。這個指令是讓 Docker 為容器分配一個虛擬終端幾，讓我們可以在容器內部進行命令操作。如果沒有使用 -t ，你就想像成沒有一個可以打指令的地方（沒有終端機）。

**/bin/bash**:代表命令解析器（Shell）。用於解釋和執行命令、腳本以及與系統交互的操作。

所以以上我們通常都會一起使用，直接使用上面範例的下者 `docker run -it ruby /bin/bash` 可以少打幾個字ＸＤ。

執行後我們可以看到實際上是進到容器裡面
![進到容器裡面](/image/dockerDay5/5_5.png)

所以我們可以在這個有 ruby 環境的容器裡進到 irb ，並執行 ruby 語法
![容器裡進到 irb](/image/dockerDay5/5_6.png)

當這時我們再開一個新的終端機，使用 `docker ps` 就可以看到，我現在正在運行的 container 了

![docker ps](/image/dockerDay5/5_7.png)

上面有說過 docker container 是有 **隔離性** 的，這邊我們可以看到上面使用 `docker ps -a` 看到已停止的容器，跟前面我們使用交互模式後，使用 `docker ps` 看到的正在執行的容器，兩者的名稱分別為 `quizzical_rhode` 跟 `lucid_rosalind` ，就可以說明他們是兩個不同的 container。

#### 命名我的 container

上面既然說到名稱，大家不會好奇那我上面那兩個名字哪來的？

當我們執行 Docker 容器而不指定名稱時，Docker 會根據一定的規則自動為容器生成一個唯一的名稱。這些自動生成的名稱通常由`形容詞＋名詞`組成。

當然我們也可以自己命名我的 container

```
docker run --name my_ruby ruby
```

![命名我的 container](/image/dockerDay5/5_8.png)
執行上方指令，可以看到我成功將我的 container 取名為 `my_ruby`

#### 背景（持久）模式執行

```
docker run -d ruby
```

這個指令是可以在背景執行，也就是不跑出執行中的 log ，若以 rails 舉例，當我們每次打開 `rails s` 時，他都會佔用一個終端機，若我想要另外開 `rails c` 時，就要多開一個終端機，那在 docker 時，使用 **-d** 即可在背景執行，不會多佔一個終端機的概念。

以上的指令都可以根據狀況組合使用，例如：

```
docker run -it --name ruby_test ruby
```

### 停止 Container

我們學會開始運行一個 Container ，就要知道如何停止它，要停止 Container 有兩種方法

#### 在容器裡停止

![在容器裡停止](/image/dockerDay5/5_9.png)
若我們使用上面學到的 `docker run -it ruby /bin/bash` 語法進到容器內，這時需要退出時可以使用

1. exit+enter
2. control+d
   打上 `exit 再按 enter 鍵` ; 或是 `按 control 鍵 + d` 可退出容器。

#### 在其他終端機停止

也可以開一個新的終端機，打上

```
docker stop <container_id_or_name>
```

至於如果不知道自己的 container_id 或 container_name ，使用前面 `docker ps` 指令可以查看正在運行的 container。

### 開始 Container

剛剛學的 `docker stop` 指令是 **停止在運行的 Container** ，所以當想要將停止的 Container 再度啟動時，則是用：

```
docker start <container_id_or_name>
```

### 刪除 Container

但是並不是每個停止的容器我們都需要再度開啟的，就像是我上面為了示範建的很多不同的 ruby container ，我確定我不會再使用了，所以我就可以使用以下語法將它刪除

```
docker rm <container_id_or_name>
```

📍 需要特別注意的是，想要刪除的 container 不能是運行中的 container ，若想刪除會出現錯誤訊息 "You cannot remove a running container"
![刪除運行中的 container](/image/dockerDay5/5_10.png)
