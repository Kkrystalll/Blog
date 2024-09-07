---
title: 使用 Ruby on Rails 串接 LINE 快速構建心理測驗
tags:
  - Line
  - Ruby
  - Rails
comments: false
toc: true
cover: /image/lineOfficialAccount/cover.jpg
categories:
  - Rails
date: 2024-09-07 23:11:02
---

# 使用 Ruby on Rails 串接 LINE 快速構建心理測驗

本文是根據我在 [2024 Hello World Dev Conference](https://hwdc.ithome.com.tw/2024) 擔任講者，並主持工作坊的經驗分享。

本篇文將會手把手教大家從建立 Rails 專案，到整合第三方 API (LINE BOT)，最後實現基於按鈕驅動的測驗互動結果。

**自備裝置**

- 電腦需先安裝好 Ruby 及 Rails
- 有 Visual Studio Code 編輯器

若還沒有安裝好 Ruby 及 Rails ，可以參考 [為你自己學 Ruby on Rails - 環境設定](https://railsbook.tw/chapters/02-environment-setup)

## 一、建立 Rails 專案及資料表

### 1. 確認 ruby 版本，確認有安裝 ruby

```
ruby -v
```

只要有看到類似 `ruby 3.3.1` ruby 後面有版本號就代表成功！

### 2. 確認 rails 版本，確認有安裝 rails

```
rails --version
```

只要有看到類似 `Rails 7.1.4` Rails 後面有版本號就代表成功！

### 3. 使用 rails 建立名為 `psychological_test` 的新專案（可以換成您喜歡的專案名稱）

```
rails new psychological_test
```

### 4. 使用 Visual Studio Code 編輯器打開此 rails 專案

![Visual Studio Code](/image/railsLinePsychological/1.png)

### 5. 在 Visual Studio Code 編輯器裡開終端機，在終端機內下 rails server，將 rails server 跑起來

![rails 專案跑起來](/image/railsLinePsychological/2.png)

```sh
# 啟動 Rails 專案
rails server
# 也可以簡寫成如下
rails s
```

若是開成功可以在終端機看到類似以下畫面
![rails s](/image/railsLinePsychological/3.png)

### 6. 打開網頁搜尋 `http://localhost:3000/` 或 `http://127.0.0.1:3000/` 可以看到畫面

![rails 專案](/image/railsLinePsychological/4.png)

### 7. 設計資料表

![question 資料表](/image/railsLinePsychological/5.png)
![result 資料表](/image/railsLinePsychological/6.png)

### 8. 使用 Scaffold 來快速生成 Question 的 CRUD

Rails 的 Scaffold 是一個命令列工具，用於自動生成基礎的 MVC（Model 、 View 、 Controller）結構，以及關聯相關的文件，如 migration、 route 和測試。當需要快速建立一個 CRUD（建立、讀取、更新、刪除）應用程式時，非常適合使用。

```
# 使用 Scaffold 來生成一個名為 Question 的 model 及其欄位等

rails g scaffold Question title option_1 option_2 value_1 value_2
```

- `rails g` : 是 rails 生成的簡寫，它是 Rails 提供的一個指令，用於產生各種文件和程式碼結構，以加速開發過程。
- `Question` : 是你要建立的 model 名稱
- `title option_1 option_2 value_1 value_2` : 在 Question model 裡面要的欄位，正常應該要寫成
  ```
  rails g scaffold Question title:string option_1:string option_2:string value_1:string value_2:string
  ```
  `title:string` 翻成中文意思就是 `欄位名稱:資料型態`，但若資料型態是 `string` ，可以簡寫成
  ```
  rails g scaffold Question title option_1 option_2 value_1 value_2
  ```

### 9. 使用 Scaffold 來快速生成 Result 的 CRUD

```
# 使用 Scaffold 來生成一個名為 Result 的 model 及其欄位等

rails g scaffold Result answers:json user_id
```

### 10. 執行數據庫遷移，將變更寫進資料庫中

```
rails db:migrate
```

### 11. 打開網網頁搜尋 `http://localhost:3000/questions` 或 `http://127.0.0.1:3000/questions` 可以看到 scaffold 生成的畫面

![question index 頁](/image/railsLinePsychological/7.png)

## 二、為資料表加上驗證 & 修改首頁

### 1. 驗證 Question 必填欄位

使用 rails model 驗證的 presence 方法，驗證欄位一定有值（不為空）[參考官方文件資料](https://guides.rubyonrails.org/active_record_validations.html#presence)

```rb
# app/models/question.rb

validates :title, :option_1, :value_1, :option_2, :value_2, presence: true
```

### 2. 驗證 Question option 欄位長度

使用 rails model 驗證的 length 方法，驗證欄位的長度最大值（最大 20 字） [參考官方文件資料](https://guides.rubyonrails.org/active_record_validations.html#length)

```rb
# app/models/question.rb

validates :option_1, :option_2, length: { maximum: 20 }
```

### 3. 將 Question index 修改成首頁

設定 root ，使我們可以在網頁搜尋 `http://localhost:3000/` 或 `http://127.0.0.1:3000/` 時，就等同於搜尋 `http://localhost:3000/questions` 或 `http://127.0.0.1:3000/questions` 的頁面

```rb
# config/routes.rb

root to: "questions#index"
```

## 三、整合 LINE Messaging API

### 1. 建立 LINE 官方帳號，拿取 Channel secret 及 Channel access token

若尚未建立帳號可以參考 [超全建立 LINE 官方帳號流程](https://kkrystalll.github.io/posts/create-line-official-account/) 文章步驟建立

### 2. 設置 LINE 的 ENV 在專案

通常在專案中，我們會將密鑰放在環境變數防止洩漏，所以我們需要手動在專案的根目錄建立一個 `.env` 的檔案來放密鑰，而此檔案需要加入 `.gitignore` ，確保此密鑰不會進入版控（但在 rails 有自動預設 `.env` 開頭的檔案都加進 `.gitignore`）

```rb
# .env

LINE_CHANNEL_SECRET=<你的 CHANNEL_SECRET>
LINE_CHANNEL_TOKEN=<你的 CHANNEL_TOKEN>
```

### 3. 安裝 line-bot-api 與 LINE Messaging API 進行互動

我們需要安裝 [line-bot-api](https://github.com/line/line-bot-sdk-ruby) 這個套件來跟 LINE Messaging API 進行互動，那在 rails 專案裡，我們需要安裝的套件會放在 `Gemfile` 這個檔案裡管理

```rb
# Gemfile

gem 'line-bot-api'
```

📍`gem 'line-bot-api'` 可以放在外層任意一行，只需注意不要放在 group 裡，因為放在外層是全部環境都需要用到的

若是加上 gem 'line-bot-api' 後需要在終端機下

```
bundle install
```

或

```
bundle
```

為了將 line-bot-api 這個 gem 安裝到 Rails 專案中。

### 4. 安裝 dotenv-rails 讀取環境變數

另外我們還需要安裝 [dotenv-rails](https://github.com/bkeepers/dotenv) 用來載入 .env 文件中的環境變數，使其可以在專案中以 `ENV['環境變數名稱']` 的格式使用

```rb
# Gemfile

group :development, :test do
  gem 'dotenv-rails', '~> 3.1', '>= 3.1.2'
end
```

記得在終端機下

```
bundle install
```

或

```
bundle
```

使這個 gem 安裝到 Rails 專案中。

### 5. 初始化 LINE Bot 客戶端

在 `config/initializers` 裡建立一個 `line_bot.rb` 的檔案，我們需要初始化一個 LINE_CLIENT 用來與 LINE 的 Messaging API 進行互動

```rb
# config/initializers/line_bot.rb

require 'line/bot'

LINE_CLIENT = Line::Bot::Client.new do |config|
  config.channel_secret = ENV["LINE_CHANNEL_SECRET"]
  config.channel_token = ENV["LINE_CHANNEL_TOKEN"]
end
```

- `require 'line/bot'`：引入了 line-bot-api 這個 gem，提供與 LINE API 互動的功能。

- `LINE_CLIENT = Line::Bot::Client.new`：建立了一個新的 LINE Bot 客戶端實體，並將它儲存到一個常數 LINE_CLIENT 中。這個客戶端會用來發送和接收來自 LINE 的訊息或事件。

- `config.channel_secret = ENV["LINE_CHANNEL_SECRET"]` 和 `config.channel_token = ENV["LINE_CHANNEL_TOKEN"]`：設定了 channel_secret 和 channel_token 密鑰，用來驗證與 LINE 伺服器的通信。
- `ENV["LINE_CHANNEL_SECRET"]` 和 `ENV["LINE_CHANNEL_TOKEN"]`: 從環境變數中讀取這些密鑰，確保這些敏感資訊不會直接暴露在程式碼中。

## 四、撰寫互動邏輯與 API 回應機制

### 1. 建立 line callback 路徑

建立一個 post 路徑，用來接收 LINE Messaging API 的 callback 訊息

```rb
# config/routes.rb

post '/line/callback', to: 'line_bot#callback'
```

### 2. 建立 line controller

在 `app/controllers` 裡建立一個 `line_bot_controller.rb` 檔案，使其繼承自 `ApplicationController`，並且定義 `callback` 這個 action

```rb
# app/controllers/line_bot_controller.rb

class LineBotController < ApplicationController
  def callback

  end
end
```

### 3. 接收 callback 訊息

參考 [官方文件](https://github.com/line/line-bot-sdk-ruby?tab=readme-ov-file#synopsis) ，驗證從 LINE 伺服器傳來的請求，確保請求是合法且來自 LINE 官方的。

```rb
# app/controllers/line_bot_controller.rb
class LineBotController < ApplicationController
  skip_before_action :verify_authenticity_token, only: [:callback]

  def callback
    body = request.body.read
    signature = request.headers['X-Line-Signature']

    unless LINE_CLIENT.validate_signature(body, signature)
      head :bad_request
        return
    end
  end
end
```

- `skip_before_action :verify_authenticity_token, only: [:callback]`: 在 Rails 中，CSRF 是一種保護機制，用來防止跨站點請求偽造攻擊，但因為現在需要允許接收來自 LINE 伺服器的 webhook 請求，所以需要讓 Rails 忽略對 callback action 的 CSRF 驗證
- `body = request.body.read`: 讀取即 LINE 伺服器傳來的內容，通常是使用者發送的訊息、加入好友等事件。
- `signature = request.headers['X-Line-Signature']`: 從 LINE 請求的標頭中取出 X-Line-Signature，這是 LINE 加密生成的一個簽名，用來確認這個請求是來自 LINE 的。
- `unless LINE_CLIENT.validate_signature(body, signature)`: 使用 LINE_CLIENT.validate_signature 方法檢查 body 和 signature 是否相符，並驗證請求的真實性。為了確保收到的事件或資料是來自 LINE 本身。
- `head :bad_request`: 回傳 HTTP 400 錯誤碼表示請求無效。

### 4. callback 邏輯

確認訊息正確後可以來寫邏輯，將從 LINE 伺服器接收到的事件來做分類

```rb
def callback
  # ...(略)
  events = LINE_CLIENT.parse_events_from(body)
  events.each do |event|
    case event
    when Line::Bot::Event::Message
      handle_text_message(event)
    when Line::Bot::Event::Postback
      handle_postback(event)
    end
  end

  head :ok
end
```

- `LINE_CLIENT.parse_events_from(body)`: 解析來自 LINE 的請求 body，將其轉換為事件陣列。
- `each`: 使用 `each` 迴圈遍歷 events 陣列中的每個事件。
- `case when`: 使用 `case when` 來判定 event 不同訊息格式，需要做什麼事
  若是 `Line::Bot::Event::Message)` 格式的話，使用 `handle_text_message(event)` 方法;
  若是 `Line::Bot::Event::Postback)` 格式的話，使用 `handle_postback(event)` 方法。
- `head :ok`: 回傳 HTTP 200 ，表示事件已成功處理。

### 5. handle_text_message 邏輯

定義 `handle_text_message` 方法，來撰寫當我們收到文字訊息時的邏輯。

```rb
def handle_text_message(event)
  user_id = event['source']['userId']
  message = event.message['text']

  if message == '心理測驗'
    start_test(user_id)
  else
    reply_message(user_id, '請輸入「心理測驗」即可開始測驗')
  end
end
```

- `user_id`: 從 event 取出發送訊息的使用者 id。
- `message`: 從 event 取出發送的訊息內容。
- `if...else`: 使用 if...else 判定接收到的訊息是否符合 `心理測驗` 四個字
  是的話執行 `start_test` 方法，並且把 `user_id` 當參數傳入;
  不是的話執行 `reply_message`方法，並且把 `user_id` 跟 `要回傳的文字內容` 當參數傳入。

### 6. reply_message 邏輯

定義 `reply_message` 方法，來撰寫我們想要回覆給使用者的訊息。

```rb
def reply_message(user_id, text)
  message = {
    type: 'text',
    text: text
  }
  response = LINE_CLIENT.push_message(user_id, message)

  if response.code != 200
    Rails.logger.error("錯誤訊息: #{response.body}")
  end
end
```

- `message`: 定義回傳的訊息格式是 text，以及回傳的訊息內容
- `LINE_CLIENT.push_message(user_id, message)`: 發送訊息給使用者
- `Rails.logger.error`: 如果沒有正確發送的話，會將錯誤訊息印在 Rails log 中

### 7. start_test 邏輯

```rb
def start_test(user_id)
  questions = Question.order(:id)

  return reply_message(user_id, '目前尚無心理測驗') if Question.none?

  result = Result.create(user_id: , answers: {})
  send_question_to_user(user_id, questions.first, result)
end
```

- `Question.order(:id)`: 找出所有的問題，並且根據 id 升序排列(由小到大)。
- `return reply_message(user_id, '目前尚無心理測驗') if Question.none?`: 如果完全沒有問題時，回傳 `目前尚無心理測驗` 訊息。
- `Result.create(user_id: , answers: {})`: 建立一個空的回答，並且寫入 user_id (之後可以延伸做成，已經做過心測的人，會直接印出答案之類的)
- `send_question_to_user(user_id, questions.first, result)`: 使用 `send_question_to_user` 發問題給使用者，並且把 `使用者 id`、`第一個問題`、`剛剛建立的空回答` 做為參數傳進去。

### 8. send_question_to_user 邏輯

```rb
def send_question_to_user(user_id, question, result)
  message = {
    type: 'template',
    altText: question.title,
    template: {
      type: 'buttons',
      text: question.title,
      actions: [
        {
          type: 'postback',
          label: question.option_1,
          data: "question_id=#{question.id}&answer=#{question.value_1}&result_id=#{result.id}"
        },
        {
          type: 'postback',
          abel: question.option_2,
          data: "question_id=#{question.id}&answer=#{question.value_2}&result_id=#{result.id}"
        }
      ]
    }
  }

  response = LINE_CLIENT.push_message(user_id, message)

  if response.code != 200
    Rails.logger.error("錯誤訊息: #{response.body}")
  end
end
```

- `message`: 建立按鈕格式的訊息，放入需要的資料 [參考官方文件](https://developers.line.biz/en/reference/messaging-api/#buttons)。

### 9. handle_postback 邏輯

```rb
def handle_postback(event)
  data = Rack::Utils.parse_nested_query(event['postback']['data'])

  result = Result.find(data['result_id'])
  question_id = data['question_id']
  answer = data['answer']

  result.answers[question_id] = answer
  result.save

  next_question = Question.where('id > ?', question_id).order(:id).first

  if next_question.present?
    send_question_to_user(event['source']['userId'], next_question, result)
  else
    reply_message(event['source']['userId'], "您的測驗結果是#{result.answers.values.join}")
  end
end
```

- `Rack::Utils.parse_nested_query(event['postback']['data'])`: 解析 LINE 的 postback 的查詢字串，並將其轉換為 Hash 物件，方便後續根據 key-value 來存取資料。
- `Result.find(data['result_id'])`: 拿取 data 裡的 result_id，並且使用 `.find` 方法找到這筆 result 資料。
- `result.answers[question_id] = answer`: 將 question_id 作為 key，answer 作為 value 寫進 result 資料裡。
- `Question.where('id > ?', question_id).order(:id).first`: 找到 id 比當前的 question_id 更大的 question ，當作下一題題目
- `if ... else`:
  如果有下一題題目 -> 使用 `send_question_to_user` 繼續傳下一題給使用者;
  如果沒有 -> 使用 `reply_message` 傳送測驗結果給使用者。
- `result.answers.values.join`: 將這筆 result 的 answers 欄位使用 `.values` 方法，拿出所有的 value ，並且使用 `.join` 方法將其組成字串。

## 五、開啟 NGROK & 設定 LINE BOT CALLBACK

### 1. 啟動 ngrok 監聽 3000 port

Ngrok 是一個工具，能將本機服務生成一個公開的網址，讓外部設備或服務（如 LINE Messaging API）能夠訪問你在本機運行的應用程式。

若還沒安裝可以參考 [官方文件](https://ngrok.com/docs/getting-started/) 安裝並且連結帳戶。

若安裝完成即可直接使用 `ngrok http 3000` 來讓 ngrok 監聽 3000 port

```
ngrok http 3000
```

會看到以下畫面

![ngrok http 3000](/image/railsLinePsychological/8.png)

我們可以拿取黃框框內 ngrok 提供的網址`5c55-2001-b011-4004-5d2a-f037-8f83-82eb-8357.ngrok-free.app`
📍 每個人生成的會不同，需換成自己生成的

### 2. 將網址放進環境變數

```rb
#.env
HOSTNAME=5c55-2001-b011-4004-5d2a-f037-8f83-82eb-8357.ngrok-free.app
```

### 3. 設定專案的 hosts

在 development 環境設置 HOSTNAME ，有公開的網址 LINE 才能正確寄送 callback

```rb
# config/environments/development.rb
config.hosts << ENV['HOSTNAME'] if ENV['HOSTNAME'].present?
```

### 4. 設定 LINE Messaging API Webhook URL

去設定頁面找到 `Messaging API` 的 `Webhook URL` 將我們的網址`https://5c55-2001-b011-4004-5d2a-f037-8f83-82eb-8357.ngrok-free.app/line/callback` 設定並儲存。

📍 `5c55-2001-b011-4004-5d2a-f037-8f83-82eb-8357.ngrok-free.app`每個人生成的會不同，需換成自己生成的
📍 `/line/callback` 路徑別忘記加上

![設定 LINE Messaging API Webhook URL](/image/railsLinePsychological/9.png)

### 5. 重開 server 並開啟 LINE 官方帳號試試

![尚未建立問題](/image/railsLinePsychological/10.png)

### 6. 建立問題

以下為我的範例問題，可以在 `http://localhost:3000/questions/new` 新增問題

| title                                                        | Option 1                           | Option 2                         | Value 1 | Value 2 |
| ------------------------------------------------------------ | ---------------------------------- | -------------------------------- | ------- | ------- |
| 當我參加一個內有一半的人不熟的聚會後，我感到                 | 認識很多新朋友，讓我充滿能量！！   | 跟不熟的人交流，我需要休息、充電 | E       | I       |
| 快要考試了但自己完全沒複習時，腦中會有什麼想法？             | 如何在時間內，規劃最大化抱佛腳效益 | 人為什麼存在？為何需要考試？     | S       | N       |
| 朋友告訴你：「我今天心情不好，所以去買了甜點吃」，你的回應是 | 買什麼甜點吃？                     | 你心情不好？怎麼了？             | T       | F       |
| 今日上班待辦清單有一大堆，你會如何規劃                       | 根據輕重緩急分配，並按計畫執行     | 做一個勾一個，中間也會適度休息～ | J       | P       |

新增成功後可以在 LINE 官方帳號看到我們自製的心理測驗 🎉🎉🎉
![自製心理測驗](/image/railsLinePsychological/11.png)

若想要看程式碼可以參考我的 [GitHub repo](https://github.com/Kkrystalll/psychologicalTest)。
