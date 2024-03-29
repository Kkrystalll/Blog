---
title: Vim 基礎常用指令
date: 2023-11-15 18:45:11
comments: false
toc: true
cover: "/image/cats.jpg"
categories:
  - Vim
tags:
  - Vim
---

## 是什麼讓我學起 vim

會知道 vim 的起源是因為之前上課的時候，看龍哥各種不需要滑鼠鍵盤一個神操作，看起來酷斃了！
但迫使我學習這些常用語法的初衷：

1. 是 `git rebase` 當我在 rebase 分支時，需要 `git rebase --continue` 繼續下一個 rebase ，這時 vscode 就會進到 vim 編輯器
2. 當我使用 AWS 跟 Docker 部署時，在終端機有時會需要使用 vim 編輯器

所以就學習了幾個常用指令今天分享給大家。

## Vim 是什麼

Vim 是從 vi 發展出來的一款**文字編輯器**，它於 1988 年釋出了 1.0 版本，到現在已經超過 35 年的歷史。

## Vim 的模式

Vim 有多種模式，其中最主要兩大的模式為：

### 正常模式（Normal Mode）：

進入 Vim 後的預設模式，可用於移動游標、刪除、複製、貼上等操作。

### 插入模式（Insert Mode）：

允許用戶輸入文字，一般編輯器都把這個設為預設模式。

## 如何打開 Vim 編輯器？

我是使用 mac ，只需要打開終端機，打上

```vim
vim <文件名稱>
```

例：我有事先準備好一個 `hello.txt` 的文件，我就可以使用

```vim
vim hello.txt
```

就可以看到使用 Vim 編輯器打開 hello.txt 的畫面
![Vim 編輯器打開檔案](/image/vim/1.png)

## 如何編輯檔案內容？

相信一開始碰到 vim 最大的問題是，怎麼打字都沒反應？？？其實是因為上面說的目前屬於預設的 **正常模式（Normal Mode）** ，如果需要在這個文件輸入文字時我們需要切換到 **插入模式（Insert Mode）**。

### 切換成插入模式（Insert Mode）

要切換成 插入模式（Insert Mode），只需要按鍵盤 `i` 或 `a` 或 `o` 任一個鍵都可以

📍 注意： 需要是英文鍵盤，如果現在鍵盤是中文按了會無效喔！

當我們按鍵盤 `i` 或 `a` 或 `o` 任一個鍵後，可以看到左下角變成 INSERT 模式，這時輸入文字就會有反應啦 🥳🥳🥳

![INSERT 模式](/image/vim/2.png)

### i a o 差在哪呢？

**i 鍵**：現在游標位置的前面
**a 鍵**：現在游標位置的後面
![vim i a o 差在哪](/image/vim/3.png)
假如我一開始的游標都放在第一行的「第」字，那我按 `i 鍵` 打字會出現的位置就是下圖的 `i` ; 反之`a 鍵` 會在 `a` 的位置（如下圖）
![vim i a o 插入位置](/image/vim/4.png)

**o 鍵**：直接換下一行新的
![vim o 鍵](/image/vim/5.png)
可以看到原本的第二行被擠下去了

## 編輯好檔案回到正常模式？

1. **esc 鍵**
2. **control 鍵** ＋ **[ 鍵**

以上兩者都可以退回正常模式，秉持著懶的天性，我個人較常使用前者。

## 如何儲存或退出檔案？

**`:w`** 是 write 儲存的意思
**`:q`** 是 quit 離開的意思

📍 以上兩個也可以合併使用
**`:wq`** 是 write & quit 儲存並離開的意思

## vim 裡的上下左右鍵

在 vim 編輯器裡也有自己的上下左右鍵
📍 原本的上下左右鍵也還是可以用喔～

**`h 鍵`** ：⬅ 左邊
**`j 鍵`** ：⬇ 下面
**`k 鍵`** ：⬆ 上面
**`l 鍵`** ：➡ 右邊

## vim 正常模式常用游標指令：

**`w 鍵`** ：跳下個字字首
**`b 鍵`** ：跳上個字字首
**`e 鍵`** ：跳當前字的字尾
**`dd 鍵`** ：連按兩次 d 鍵，就可以不用進到插入模式刪除該行
**`shift 鍵`** ＋ **`) 鍵`** ：跳到這行的句首
**`shift 鍵`** ＋ **`^ 鍵`** ：跳到這行的句首
**`0 鍵`** ：跳到這行的句首(有了這個前兩個我直接忘ＸＤ)
**`shift 鍵`** ＋ **`( 鍵`** ：跳到這行的句尾
**`shift 鍵`** ＋ **`$ 鍵`** ：跳到這行的句尾
**`gg 鍵`** ：跳到文件最開頭

以上就是我整理覺得最常用到的 vim 指令們！若是大家有機會用到 vim 編輯器可以試試喔～
