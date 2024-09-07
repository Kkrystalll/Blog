---
title: 超全建立 LINE 官方帳號流程
tags:
  - Line
  - 設定
comments: false
toc: true
cover: /image/lineOfficialAccount/cover.jpg
categories:
  - LINE
date: 2024-08-22 20:40:03
---

## 一、去 LINE 官方帳號官網登入或註冊帳號

1. [首頁](https://tw.linebiz.com/) 點擊右上角 [免費開設帳號](https://tw.linebiz.com/account/)

   ![免費開設帳號](/image/lineOfficialAccount/1.png)

2. 在「LINE 官方帳號」區塊點擊 [免費開設帳號](https://account.line.biz/login?redirectUri=https%3A%2F%2Faccount.line.biz%2Foauth2%2Fcallback%3Fclient_id%3D18%26code_challenge%3DFCA3UR74dteRWFHWN0geBTNxyCrxTnCi_xEMJeiD8As%26code_challenge_method%3DS256%26nonce%3DHF3FUJOrb1ipkC7FV8XckVpcStWHPvJK%26path%3D%25252Flogin%25253Fscope%25253Dphone%26redirect_uri%3Dhttps%253A%252F%252Fentry.line.biz%252Fapi%252Foauth2%252FbizId%252Fcallback%26response_type%3Dcode%26state%3Dkcyhly7MmN7cdr1jUCsZV1ovxDvZmlE8) 按鈕

   ![免費開設帳號](/image/lineOfficialAccount/2.png)

3. 選擇登入方式

- **使用 LINE 帳號登入**：使用舊有的 LINE 帳號登入
- **使用商用帳號登入**：使用舊有的商用帳號登入
- **建立帳號**：創建新帳號（本文選擇此方式）

  ![建立帳號](/image/lineOfficialAccount/3.png)

4. 選擇建立帳號
   選擇使用電子郵件帳號註冊，並填入電子郵件
   ![電子郵件帳號註冊](/image/lineOfficialAccount/4.png)
   ![填入電子郵件](/image/lineOfficialAccount/5.png)

5. 去你填入的電子郵件收件夾點擊按鈕或複製連結網址
   ![驗證](/image/lineOfficialAccount/6.png)

6. 輸入姓名密碼
   ![輸入姓名密碼](/image/lineOfficialAccount/7.png)
   ![確認](/image/lineOfficialAccount/8.png)
7. 完成後可以看到右上角有自己的名字，代表已登入管理頁面

   ![已登入管理頁面](/image/lineOfficialAccount/9.png)

## 二、建立 LINE 官方帳號

1. 點擊「建立 LINE 官方帳號」
   ![建立 LINE 官方帳號](/image/lineOfficialAccount/10.png)

2. 使用此帳號登入
   ![登入](/image/lineOfficialAccount/11.png)

3. 需先進行簡訊驗證
   ![簡訊驗證](/image/lineOfficialAccount/12.png)
   ![認證](/image/lineOfficialAccount/13.png)

4. 填入帳號資訊
   ![登入公司店舖資訊](/image/lineOfficialAccount/14.png)

5. 確認輸入內容
   ![確認輸入內容](/image/lineOfficialAccount/15.png)

6. 申請完成會獲得官方帳號 ID
   ![官方帳號 ID](/image/lineOfficialAccount/16.png)

7. 點擊回到管理頁面按鈕，會先需要同意政策
   ![同意政策](/image/lineOfficialAccount/17.png)

8. 可以使用其他帳號掃 QRcode 加入好友，或是搜尋 `@042okcst`，並回到主頁
   ![QRcode](/image/lineOfficialAccount/18.png)
   ![回到主頁](/image/lineOfficialAccount/19.png)

9. 看到主控台
   ![官方帳號 ID](/image/lineOfficialAccount/20.png)

10. 加入好友後看到畫面
    ![加入好友](/image/lineOfficialAccount/21.jpg)

📍 到這邊為止已經算是完成建立 LINE 官方帳號，後續步驟為後續為了串接 LINE 的第三方 API 所需要做的基本設定。

## 三、開啟 Messaging API 服務

1. 點選右上角「設定」按鈕
   ![設定](/image/lineOfficialAccount/22.png)
2. 點選左方 sidebar 「Messaging API」按鈕
   ![Messaging API](/image/lineOfficialAccount/23.png)
3. 點選「啟用 Messaging API」 按鈕
   ![啟用 Messaging API](/image/lineOfficialAccount/24.png)
4. 根據步驟填入資訊
   ![填開發者資訊](/image/lineOfficialAccount/25.png)
   ![確認資訊](/image/lineOfficialAccount/26.png)
   ![選擇服務](/image/lineOfficialAccount/27.png)
   **選填可不填直接按確定**
   ![隱私權政策](/image/lineOfficialAccount/28.png)

   ![啟用服務](/image/lineOfficialAccount/29.png)

5. 啟用完成可以看到自己的 `Channel ID` 以及 `Channel secret` ，需記下來
   ![Channel ID. Channel secret](/image/lineOfficialAccount/30.png)

## 四、找到 LINE Channel access token

1. 點擊「LINE Developers」按鈕
   ![LINE Developers](/image/lineOfficialAccount/31.jpg)

2. 點選右上方「Console」按紐
   ![Console](/image/lineOfficialAccount/32.png)
3. 選擇剛建立的官方帳號
   ![選擇官方帳號](/image/lineOfficialAccount/33.png)
4. 點選「Messaging API」按鈕
   ![Messaging API](/image/lineOfficialAccount/34.png)
5. 可以看到 Channel access token 區塊，點擊「Issue」按鈕
   ![產生 Channel access token](/image/lineOfficialAccount/35.png)
6. 生成 `Channel access token` ，需記下來
   ![Channel access token](/image/lineOfficialAccount/36.png)

以上為完整建立 LINE 官方帳號，以及為了後續串接 LINE 的第三方 API 所需要做的基本設定，實際辦過發現小流量的官方帳號可以免錢真的好佛啊～

2024/09/07 更新：

LINE 自 2024/09/03 強制啟用雙重驗證，我使用我先前已經註冊的帳號登入，會被引導到雙重驗證的頁面
![雙重驗證](/image/lineOfficialAccount/37.png)
只需要依照步驟去收驗證碼信，填入驗證碼即可成功驗證並登入!
![驗證碼信](/image/lineOfficialAccount/38.png)