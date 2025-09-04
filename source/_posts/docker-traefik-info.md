---
title: Docker - 了解 Traefik 反向代理伺服器
tags:
  - Docker
  - 部署
  - Traefik
comments: false
toc: true
cover: /image/docker.webp
categories:
  - Docker
  - 部署
  - AWS
  - Traefik
date: 2023-10-08 10:27:38
---

昨天已經部署完了就達成目的啦，怎麼我今天還出現？因為鐵人賽規定要 30 天啊！(開玩笑的)，其實是因為目前大概只能說是完成部署的 7 成，不知道大家有沒有發現，我們一直以來輸入的網址，無論是 DNS

```
http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000
```

或是 IP Address

```
http://52.199.213.167:3000
```

兩者的網址都是 HTTP ，但其實在瀏覽器也會顯示 HTTP 是個 `不安全` 的通訊協議

![搜尋 http 網址](/image/dockerDay23/23_1.webp)

正常應該要是 HTTPS 的安全網址才是正確的，那我們應該要如何將網址從 HTTP 變成 HTTPS 呢？

## 將網址從 HTTP 變成 HTTPS 的步驟

### 1. 取得 SSL/TLS 憑證：

在一個受信任的憑證授權單位（CA）（例如 Let's Encrypt、Comodo、DigiCert 等）來取得 SSL/TLS 憑證。有些 CA 提供免費的證書，有些可能需要付費。

### 2. 安裝證書：

在您的網頁伺服器上安裝 SSL/TLS 憑證。這通常涉及將憑證檔案和私鑰檔案放置在伺服器上的特定位置。

### 3. 配置伺服器：

修改你的網頁伺服器設定檔來啟用 HTTPS。
設定檔應包括`指定 SSL/TLS 憑證的路徑`、`設定伺服器以在 443 port 上監聽 HTTPS 請求`、`啟用 SSL/TLS 協定` 等。

### 4. 重新啟動伺服器：

重新啟動伺服器以應用新的設定。

### 5. HTTP 到 HTTPS 自動轉址：

設定伺服器在存取 HTTP 網址時，自動轉址到 HTTPS。這樣，無論使用者輸入是 HTTP 還是 HTTPS，它們都會被自動轉址到安全的 HTTPS 連線。

以上是我簡化後的步驟就有五個，這麼麻煩的事情如果使用反向代理伺服器，如： Traefik 就可以全部簡化，那我還不快來認識 Traefik !!!

## 使用 Traefik 將 HTTP 變成 HTTPS ，可以做到哪些事？

### 自動 SSL/TLS 憑證管理：

Traefik 可以與 Let's Encrypt 或其他證書頒發機構合作，自動獲取、更新和配置 SSL/TLS 證書。這樣就不用需要手動管理證書有沒有過期啥的，它會自動處理，就像請一個小秘書一樣方便 🤩。

### HTTP 到 HTTPS 自動重新導向：

Traefik 可以輕鬆設定 HTTP 到 HTTPS 的自動轉址，確保所有流量都是加密的。

### 負載平衡與路由：

Traefik 支援負載平衡和路由配置，可將流量分發到多個負載服務，實現高可用性和容錯能力。

### 容器環境友善：

Traefik 特別適用於容器化環境，例如 Docker ，他都這樣說了我還不用？

### 簡化配置：

Traefik 的配置相對簡單，可以將上述複雜的步驟簡化還不夠簡單嗎？

總之，Traefik 簡化了將 HTTP 網站升級為 HTTPS 的過程，並提供了自動化、簡單配置和容器環境友善的功能。另外，Traefik 還提供了強大的反向代理和負載平衡功能，可增強您的應用程式的效能和安全性。

![Traefik 配置](/image/dockerDay23/23_2.webp)

我們可以參考 [官網](https://doc.traefik.io/traefik/) 上的圖片，其實我就是把 Traefik 想像成交通警察阿北，如果迷路走到 HTTP 他會幫你導正路線到 HTTPS ; 如果這道車特多，他會引導你去隔壁道比較順暢。

所以明天我們就要來實際使用 Traefik ，來將我們的網址變成安全的網址，那就明天見～
