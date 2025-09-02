---
title: Docker - Docker Networks(網路)容器與容器間的橋樑
tags:
  - Docker
  - 部署
  - network
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
date: 2023-09-21 02:31:03
---

前一章有說 Docker Container (容器)是有隔離性的，也就是互不影響，但這樣的話，我們應該要如何連結兩個容器呢？這時候 Docker Network (網路)就出場啦！ Docker Network (網路)是 Docker Container (容器)與 Container (容器)之間 ; 以及 Container (容器)與外部容器、網路或服務之間連接通訊的橋樑。Docker 提供了不同類型的網絡，使容器能夠連接到這些不同類型的網絡。

以下就來介紹三種使用 Docker 時，一定會遇到的基礎網路模式：

### Bridge Network (橋接網絡)：

bridge 是在 Docker 裡預設的網路。在 bridge 網路模式中，每个 Container (容器)都分配一个獨立、不重複的 IP 地址，Container (容器)可以通过这个 IP 地址相互聯絡。容器可以通過一個主機的 NAT（網絡地址轉換）訪問外部網絡。

> 容器可以通過一個主機的 NAT（網絡地址轉換）訪問外部網絡

我在看上述這句話的時候覺得看了沒有很懂，所以了解了一下，再以自己的話加以解釋：

上面有說到每個 Container (容器)都會分配一个獨立、不重複的 IP 地址，而這些 IP 地址是私有的，只能在 Container (容器)內找到，無法跟 Container (容器)外部聯絡。所以當 Container (容器)需要與外部聯絡時， Docker 主機會成為 Docker Network(網路)的 NAT ，全名為 Network Address Translation（網路地址翻譯），也就是可以把 Docker 主機想像成翻譯機，將 Container (容器)內的請求，翻譯成外部可以聽得懂的語言，然後將外部回來的 response 再翻譯回給 Container (容器)。

### Host Network（主機網路）：

當 Container (容器)使用 Host Network 模式時，代表它們與主機共享同樣的網絡接口和設定，也就是與主機具有相同且公開的 IP 地址。這意味著 Container (容器)可以像主機 (Host) 一樣直接使用主機上的網絡資源，所以並不需要使用 NAT 這個翻譯機來翻譯。

另外通常，在 Docker 中，容器的 port (端口)可以對應到主機的某個 port (端口)，以便從外部網絡可順利對應到 Container (容器)中的服務。但在主機網絡模式下，容器不需要 port (端口)對應。容器的網絡設定與主機上的網絡設定一樣，它們可以使用相同的 port (端口)。

### None Network（無網路）：

在 None Network（無網路）模式下， Container (容器)不會連接到任何網絡，因為根本沒有網絡接口。
None Network（無網路）適用時機會是比較特殊的情況，例如：我希望 Container (容器)完全隔離並且沒有任何網絡訪問權限。

總結來說，因為 Host Network（主機網路）可以直接以主機本人身份加進去網路，就像一般人出國需要翻譯，但是當翻譯官本人出國時，因為本來就會當國語言，他就不需要再另外請翻譯，因為他自己就聽得懂。所以在一些情況下使用 Host Network（主機網路）是非常有用的。但是需要特別注意，因為是可以直接以主機身份操作，這樣可能會影響 Container (容器)網路的隔離性和安全性，所以不能濫用。
而 None Network（無網路）就別說是出國了，根本是把人丟到一座無人島，我 Container (容器)幾乎等於與世隔絕，無法與外界聯繫。

在了解三種使用 Docker 時，一定會遇到的基礎網路模式後，不曉得大家會不會跟我一樣充滿問號，說這麼多，啊我到底該如何知道我現在有哪些網路？以及我的每一個 Container (容器)分別使用哪些網路？最後是我應該如何建立一個屬於自己的網路，以及將 Container (容器)使用我建立的網路？

### 查看本機 docker 網路列表

```docker
docker network ls
```

![看本機 docker 網路列表](/image/dockerDay6/6_1.webp)

根據 [官網](https://docs.docker.com/engine/reference/commandline/network_ls/) 的圖片，我們可以看到在最一開始，使用 `docker network ls` 可以看到有三種名稱的網路：`bridge`、`host`、`none` 這三個是 docker 基礎預設的網路。

### 我的每一個 Container (容器)分別使用哪些網路

要想知道任一個 Container (容器)的詳細資訊，例如：使用哪個網路、環境變數有哪些等等時，可以使用

```docker
docker inspect <container_id_or_name>
```

![容器的詳細資訊](/image/dockerDay6/6_2.webp)

圖片是擷取有關網路部分的資訊，實際上還有很多其他資訊喔！
根據圖片我們可以得知，這個 Container (容器)是使用預設的 Bridge Network (橋接網絡)，另外可以看到更多詳細的 ID 或是 IP 位置等等。

### 建立及使用我所建立的網路

那我的這個容器不想使用預設的 Bridge Network (橋接網絡)，我想要有自己的網路，可以使用

#### 建立網路

```docker
docker network create <network_name>
```

![建立網路](/image/dockerDay6/6_3.webp)

當建立成功可以看到一個獨一無二的 SHA 值
我們再次使用 `docker network ls` 查看，可以看到 my-net 已經成功被建立
![docker network ls](/image/dockerDay6/6_4.webp)

#### 將我的 container natwork 換成特定 natwork

首先先使用 `docker ps` 看有哪些正在運行的 Container (容器)
![docker ps](/image/dockerDay6/6_5.webp)

##### 連接 Network（網路）與 Container (容器)

```docker
docker network connect <network_name> <container_id_or_name>
```

![連接 Network 與 Container](/image/dockerDay6/6_6.webp)
這邊執行成功並不會有輸出，所以看到如圖片的空白行是正常的！

##### 斷開 Network（網路）與 Container (容器) 的連接

```docker
docker network disconnect <network_name> <container_id_or_name>
```

![斷開連接](/image/dockerDay6/6_7.webp)
同上成功並不會有輸出。
這時再使用 `docker inspect <container_id_or_name>` 可以看到這個容器，已經沒有連接 my_net

#### 移除 Network（網路）

```docker
docker network rm <network_name>
```

![移除 Network](/image/dockerDay6/6_8.webp)

這時再使用 `docker network ls` 可以看到已經沒有 my_net 這個網路

今天了解了 Docker Network(網路) 的基本語法及觀念，明天來繼續深入了解昨天有小提過的 Docker Tag。
