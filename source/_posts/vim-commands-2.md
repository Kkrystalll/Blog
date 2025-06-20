---
title: Vim 基礎常用指令第二彈
date: 2025-06-10 21:45:11
comments: false
toc: true
cover: "/image/cats.jpg"
categories:
  - Vim
tags:
  - Vim
---

時光飛逝一年半就這樣過去了， Vim 會的指令依然屈指可數，趕快再來多學幾個新的指令吧！

## 快速頁面導航

`Ctrl + f`：向下一頁 (如果你內文不夠長的話，就會變成跳到當前頁最後一行)

`Ctrl + b`：向上一頁

`Ctrl + d`：往下滑半頁 (terminal 高度的一半)

`Ctrl + u`：往上滑半頁

`G`：跳到文件最後一行

`{數字}G`：跳到指定行，例如 10G 跳到第 10 行

## 分割視窗

`:sp`：水平分割當前檔案的視窗

`:sp [文件名稱]`：水平分割新文件與當前檔案視窗

`:vsp`：垂直分割當前檔案的視窗

`:vsp [文件名稱]`：垂直分割新文件與當前檔案視窗

`Ctrl + w + w`：切換到另一個視窗

`Ctrl + w + h/j/k/l`：向左 / 下 / 上 / 右移動視窗

`Ctrl + w + =`：使所有視窗等寬等高

## 刪除與修改

`x`：刪除當前游標下的字

`X`：刪除當前游標前的字

`dw`：刪除從當前位置到下一個單詞開始

`d$` 或 `D`：刪除從當前位置到行尾

`dd`：刪除整行

`cw`：修改從當前位置到下一個單詞開始 (當前字刪掉，會進入插入模式)

`c$`：修改從當前位置到行尾 (游標以後這行的字刪掉，會進入插入模式)

`cc`：修改整行 (整行刪掉，會進入插入模式)

## 進階修改

`ci"`：修改引號內內容 (會進入插入模式)

`ca(`：修改「包含」括號的整個內容 (會進入插入模式)

`ct.`：修改現在游標位置，直到遇到第一個句點 (會進入插入模式)

## 複製與貼上

`yy`：複製整行

`y$`：複製從當前位置到行尾

`p`：在光標後貼上

`P`：在光標前貼上

可以搭配使用

`yy + p`：複製這行，在下一行貼上
