---
title: 使用 Google Script 來管理 Gmail
date: 2019-09-13 02:41:51
categories: Uncategorized
tags:
- gs
- organize
toc: true
---

[Google Apps Script](https://developers.google.com/apps-script/) 是 Google 自己出的 Script，可以拿來串接 Google 各項服務的 API

上學期期末的時候其實就聽過系上社團學長說他有用這個在管理他的 Gmail，當然那個時候也是第一次聽過有 Google Script 這種東西

不過期末實在太忙了就沒有研究，然後放暑假 email 們也少少的所以也沒有想到

直到最近開學，直接被各種學校公告學習系統 email 轟炸，熊熊覺得可以來研究一下了

<!-- more -->

亂摸了幾天總算給我摸出了個小功能ˊˇˋ

就是每天固定時間將一些公告帳號寄來的郵件清掉

## 建立一個 Script

首先要先到 Google Drive 新增一個 Script 檔
找不到的話點 `連結更多應用程式` 裡面會有

![](https://i.imgur.com/slQIK05.png)

打開之後沒意外會是一個專案，裏面有一個`.gs`檔

## Gmail API

以下是我`.gs`檔裡面的內容

```javascript
function deleteThreads() {
  var searchRules = 'in:inbox -in:starred has:nouserlabels from:'
  var toDelete = ['sport@cc.nctu.edu.tw', // 交大體育室
                  'nctuannounce@nctu.edu.tw', // 交大公告
                  'cdcp_notice@nctu.edu.tw'] // E3公告
  for (var i = 0; i < toDelete.length; i ++) {
    var threads = GmailApp.search(searchRules + toDelete[i]);
    for (var j = 0; j < threads.length; j ++) {
      threads[j].moveToTrash();
    }
  }
}
```

簡單翻譯一下，`toDelete` array 中是我想刪除的郵件們的 sender
透過 `searchRules` 和 `toDelete[i]` 可以組成一個搜尋條件
這個搜尋條件和我們平常打在 Gmail 搜尋框跑出來的搜尋結果一樣
![](https://i.imgur.com/8JIFNAb.png)
詳細資訊可以看看 [可搭配 Gmail 使用的搜尋運算子](https://support.google.com/mail/answer/7190?hl=zh-Hant)

至於 `searchRules` 的意思：
- `in:inbox`: 在收件夾中，封存的不算
- `-in:starred`: 未加星號（\- 代表 not）
- `has:nouserlabels`: 沒有自訂 label

接著將搜尋條件塞入 `GmailApp.search()`
結果存進 `threads`，也就是想刪除的 email 對話串
最後透過 `threads[j].moveToTrash()` 將對話串一個一個刪掉

大概就是這樣
想寫其他功能還要再多研究 [Gmail API](https://developers.google.com/apps-script/reference/gmail/)

## Time Trigger

有了刪郵件的功能
再來就是新增一個 Time Trigger 來幫我每天刪郵件了

按`編輯 > 現有專案的啟動程序` 就會跳到 Developer Hub 的頁面
接著按 `新增觸發條件`
之後就是愉快的圖形介面設定了
大概就是選擇某個 function 還有它的發動時間及頻率
儲存之後就大功告成了！
