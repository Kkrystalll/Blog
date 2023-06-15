---
title: Ruby - 到底 if...else 好？還是 case...when 好？
date: 2023-06-15 18:45:11
comments: false
toc: true
cover: "/image/caseWhen.png"
categories:
  - Ruby
tags:
  - Ruby
  - if...else
  - case...when
---

## 寫在前面

上個月與同事互相 code review 時，我寫了一個 if...else 判定式，同事建議可以將其改成 case...when ，當時的我覺得兩者好像大同小異，但我還是照者修改了，但還是好奇的問了同事，為何 case...when 比較好啊？經過一番討論，發現我們自己也講不出具體的原因，所以決定來稍做研究 (ﾉ>ω<)ﾉ

## 先寫結論

### if…else 特性

> 適合在需要根據條件執行不同或進行多個條件判斷的情況下使用。

### case…when 特性

> 適合在需要對一個變數或表達式的許多可能值進行搭配時使用。

## if...else 改寫 case...when

以下範例是簡單使用 if...else 改寫 case...when ，可以使用其先稍做暖身。

範例ㄧ：

### 使用 if...else

```ruby
gender = "女"

if gender == "女"
  puts "我是女生"
elsif  gender == "男"
  puts "我是男生"
else
  puts "我是其他"
end
```

### 使用 case...when

```ruby
gender = "女"

case gender
when "女"
  puts "我是女生"
when "男"
  puts "我是男生"
else
  puts "我是其他"
end
```

📍 補充：
在 ruby 裡 case...when 看似跟 JavaScript 的 `switch...case` 很相似，但兩者最大的不同是 ruby 裡 case...when 不需要 break ，當其符合條件時會自動跳出這個 case...when 不會繼續執行下一個 when ; 反之，在 JavaScript 的 switch...case 若沒有在每個 case 語句的結尾使用 break ，則他會繼續執行下一個 case 語句，直到遇到
break 或是執行結束，才會真的結束。

---

## case...when 到底怎麼比對的呢？

在 ruby 的 case...when 其實就是用 `===` 在做判定的，至於是誰 === 誰，讓我們用範例看看吧！

範例二：

### 使用 if...else

```ruby
value = 5

if value.is_a?(Integer)
  puts "It's an integer"
elsif value.is_a?(String)
  puts "It's a string"
elsif value.between?(1, 10)
  puts "It's within the range 1 to 10"
else
  puts "It's something else"
end
```

### 使用 case...when

```ruby
value = 5

case value
when Integer
  puts "It's an integer"
when String
  puts "It's a string"
when 1..10
  puts "It's within the range 1 to 10"
else
  puts "It's something else"
end
```

答案就是 `when === case` ，以範例二的第一個 when 為例，其實就是 `Integer === value` 為 true 來做比對。
不知道有沒有人跟我一樣好奇，怎麼能這麼肯定？說不定是 `case === when` 呀，那我們可以直接拿上述 Integer 跟 value 試試

```ruby
Integer === value # true
value === Integer # false
```

相信大家看了就很明顯可以知道兩者差異。

---

## case...when 的多重判斷（ ~~多重宇宙？！~~ ）

事實上在 ruby 的 case...when 可以說是 if...else 的 Syntax sugar （語法糖衣），除了都可以用於條件判斷外， case...when 有個特點之一是可以做多重判斷

範例三：

### 使用 if...else

```ruby
if day=="Monday" || day=="Tuesday" || day=="Wednesday" || day=="Thursday" || day=="Friday"
  puts "上班日"
elsif day=="Saturday" || day=="Sunday"
  puts "休假日"
else
  puts "格式錯誤"
end
```

### 使用 case...when

```ruby
day = "Monday"

case day
when "Monday", "Tuesday", "Wednesday", "Thursday", "Friday"
  puts "上班日"
when "Saturday", "Sunday"
  puts "休假日"
else
  puts "格式錯誤"
end
```

像是這種多重判斷，我就會推薦使用 case...when ，與語法更簡潔易讀。

---

## 比對範圍大小用什麼好呢？

範例四：

### 使用 if...else

```ruby
score = 85

if score >= 90
  puts "优秀"
elsif score >= 80
  puts "良好"
elsif score >= 70
  puts "中等"
elsif score >= 60
  puts "及格"
else
  puts "不及格"
end
```

這邊請大家改寫成 case...when 請問會怎麼寫呢？

### 使用 case...when

```ruby
score = 85

case score
when score >= 90
  puts "优秀"
when score >= 80
  puts "良好"
when score >= 70
  puts "中等"
when score >= 60
  puts "及格"
else
  puts "不及格"
end
```

不知道大家會不會跟我第一直覺想的一樣，寫成如上的判定式，但其實這會跟我們預期印出 `"良好"` 的結果不同，而是印出 `"不及格"` ，其實是因為剛剛上面有說是用 === 做比對， 那 `score >= 80 === score` 的結果想必是 false ，所以他自然會印出 else 的`"不及格"`。

### 正確的使用 case...when 改寫

```ruby
score = 85

case
when score >= 90
  puts "优秀"
when score >= 80
  puts "良好"
when score >= 70
  puts "中等"
when score >= 60
  puts "及格"
else
  puts "不及格"
end
```

📍 補充：
已範例四來說，在 RuboCop 會建議使用 if...else 來替換 case...when 原因是，我們根本 case 不需要將 score 變數與每個 when 條件進行比較，這時他就會更推薦使用 if...else 更易讀。

## 總結

### if...else 的特點：

- 可以處理更複雜的條件判斷，因為可以使用比較運算符（例如 >、<、== 等）以及邏輯運算符（例如 &&、|| 等）。
- 每個 if 語句僅處理一個條件，可以有多個 elsif 來處理不同的情況，也可以包含 else ，處理其他未滿足的情況。
- 適用於需要根據不同條件執行不同程式碼的情況。

### case...when 的特點：

- 主要用於根據某個值進行比對，而不是條件判斷。
- 每個 when 條件可以包含多個值或區間。
- 可以使用其他 Ruby 表達式作為 when 條件。
- 適用於需要根據不同值進行比對並執行相應程式碼的情況。
- 可以提供更清晰、簡潔和易於擴展的結構，特別是在需要處理多個情況時。

參考資料：
https://www.akshaykhot.com/ruby-switch-statement/
[MDN JavaScript switch...case](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/switch)
