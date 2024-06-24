---
title: 從原理到精通的 React 全攻略課程
date: 2023-02-19 18:58:45
comments: false
toc: true
cover: "/image/reactCourse.jpg"
categories: 課程心得
tags:
  - React
  - 心得
  - 課程
---

## 參加 React 完全進階攻略 動機

在五倍參加完 ASTRO camp 幸運的很快速找到工作，但第一份工作居然是要寫 React ？！因為看到五倍也有開設 **React 完全進階攻略** 課程，加上在 ASTRO camp 其間有上過一堂奶綠老師的基礎 React 課程，喜歡老師在課堂中會穿插一點笑話，讓整體課程不會過於沉悶的教學風格，於是乎我又再度回到五倍的懷抱，利用假日來上奶綠老師的 **React 完全進階攻略** 課程，如此一來我就可以平日上班實作 React 專案，假日跟著老師了解 React 原理，以及學習怎樣叫做漂亮的 Pure Function 。

## 課程內容貼和業界

雖然課程名稱是 **React 完全進階攻略** 但老師上課會由淺入深的帶入，從 Components 如何撰寫，到 Testing Library 測試，所以相信跟我一樣對於 JavaScript 有基礎，但對 React 幾乎是零基礎的人，也可以跟得上進度，加上我自認自己滿幸運的，一起上課的學生們大部分都跟我一樣工作上有運用到 React ，除了老師上課舉例是業界工作上實際會遇到的雷外，同學們也會提問自己專案上遇到的問題，讓這堂課幾乎可以說是完全貼合現在業界的需求。

## 怎樣的人適合這門課程

在工作與學習並行的情況下，我每週學習到新的或是更好的寫法時，都可以立即運用在專案上，抑或是遇到一些困難點，也可以跟老師討論一下大方向，非常慶幸自己在剛上工即報名課程，大大縮短了我上工時的陣痛期。我覺得這門課程除了適合對 React 有興趣的人外，更適合目前工作上也會用到 React 的人，可以在課程中發現自己寫 code 的盲點，或是了解更好的做法。

以我自己為例：像是簡單的 Modal 開關，在上課前的我絕對是利用 `useState` 控制開合，再在最外層 `useEffect` 時打 API 資料，內層需要資料時再一層一層將 props 傳入，實際上這個做法會導致傳入一堆 props ，讓自己距離 Pure Function 更加遙遠（因為層層互相影響），且平行層級的 `Components` 用此方法非常難拿到資料；但在上課後，學到了 `useContext` 、 `useReducer` 等更進階的用法原理時，我就像是打通任度二脈，可以節省很多參數傳來傳去的弊端!

在學會更進階的用法後，就變成如下的範例，如此一來程式碼可讀性提高，且光看英文大概都可以知道是在控制不同 Modal 開關！

```react
import { BuyListContext } from "components/buyer/list"

<BuyListContext.Provider value={{ state, dispatch }}> // 在最外層的元件包 BuyListContext.Provider 可以使內層的元件在不用傳 props 的狀況下拿到 value
    // 元件內容...
</BuyListContext.Provider>
```

```react
import { useContext } from "react"
import { Modal } from "components/shared" // 客制 modal 元件
import { SubmitButton } from "components/shared" // 客制 SubmitButton 元件
import { BuyListContext } from "components/buyer/list"

const CancelHintModal = ({ open }) => {   // 用 open 傳入 modal 開合的布林值
  const { dispatch } = useContext(BuyListContext)

  const handleBackOrderDetial = () => {
    dispatch({ type: "CANCEL_HINT_MODAL_SWITCH", payload: false })
    dispatch({ type: "ORDER_DETAIL_MODAL_SWITCH", payload: true })
  }

  const handleToCancelReason = () => {
    dispatch({ type: "CANCEL_REASON_MODAL_SWITCH", payload: true })
    dispatch({ type: "CANCEL_HINT_MODAL_SWITCH", payload: false })
  }

  return (
    <Modal open={open} showIcon={false}>
      <h3>取消訂單提醒</h3>
      <p>為保障買賣雙方的權利，若商品為已發貨，將不能取消訂單。</p>
      <div>
        <SubmitButton value="取消" isHollow={true} atClick={handleBackOrderDetial} /> // 若點下取消，則關閉此 modal ; 且開啟訂單詳細資訊 modal
        <SubmitButton value="繼續" atClick={handleToCancelReason} /> // 若點下繼續，則開啟取消原因 modal ; 且關閉此 modal
      </div>
    </Modal>
  )
}
```

## React 完全進階攻略 課程最讓我覺得物超所值的是

因為奶綠老師本職就是業界的資深工程師，除了上述說到老師會舉實際上工作會遇到的雷外，更會介紹很多實際 React 專案會用到的套件或是小工具，讓我們可以更快上手。但撇除以上幾點，我自己最喜歡這堂課的最關鍵點是，每堂課都會有個回家作業，當交作業後，老師會親自幫你 code review ，畢竟要寫得出功能或許不難，但要寫出好的 code ，對現階段的我來說還是非常不足的，而參加 **React 完全進階攻略** 除了可以學到 React 的運用及原理，附加價值更是可以獲得業界資深工程師親自 code rewiew，對我這個小菜雞來說已經太超值！以上參加課程的心得希望可以幫助到，對 React 有興趣，或是正在猶豫的大家～
