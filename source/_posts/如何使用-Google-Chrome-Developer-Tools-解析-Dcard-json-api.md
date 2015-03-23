title: "如何使用 Google Chrome Developer Tools 解析 Dcard json api"
date: 2015-03-07 10:31:02
tags:
---

本文要介紹如何使用 Google Chrome Developer Tools 觀察 api 呼叫狀況，得到如何呼叫 Dcard 公開的 JSON api.

### 如何開啟

按下 設定 > 更多工具 > 開發人員工具
![/img/how-to-use-google-developer-tools-parsing-dcard/sWgkgIL.png](/img/how-to-use-google-developer-tools-parsing-dcard/sWgkgIL.png)


接這就出現工具列了！第一個畫面看到的是 Elements，會看到大大的 Dcard 顏文字。這頁主要是顯示 HTML 的 DOM 樹
![/img/how-to-use-google-developer-tools-parsing-dcard/rHkuFyy.png](/img/how-to-use-google-developer-tools-parsing-dcard/rHkuFyy.png)


今天的主角是 Network，觀察網路 API 傳輸狀況。因為網頁開好才開起來，這裡只會看到 [.map 檔](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)的 request 
![/img/how-to-use-google-developer-tools-parsing-dcard/uUoe8pu.png](/img/how-to-use-google-developer-tools-parsing-dcard/uUoe8pu.png)


重新整理頁面之後就可以看到所有 Request，可以看到這個網頁有數百個 request，右邊有選項可以選擇只看特定類型的。分別為：HTML 檔、CSS 樣式、圖片、影音、Script 腳本、XHR (Ajax 請求)、字體、WebSockets、其他
![/img/how-to-use-google-developer-tools-parsing-dcard/kUpxP10.png](/img/how-to-use-google-developer-tools-parsing-dcard/kUpxP10.png)

由於 Dcard 是用 Single Page Application 架構，所有 request 都是用 Ajax 方式得到，所以大力的點下 XHR 可以看到所有 JSON api 請求。Request 主要分成 /partial 跟 /api 兩大類，/partial 是該頁面需要的 HTML 片段，/api 是該頁面需要的資料。
![/img/how-to-use-google-developer-tools-parsing-dcard/bWtS3Mz.png](/img/how-to-use-google-developer-tools-parsing-dcard/bWtS3Mz.png)

大力點下我們關注的 /api，這裡選擇解析 /api/forum/all/1，可以在 http://www.dcard.tw/f 看到這個 request。第一個看到的頁面是 header，可以搜尋一些網路上的 [HTTP Header 詳解](http://kb.cnblogs.com/page/92320/)了解如何送 data。
![/img/how-to-use-google-developer-tools-parsing-dcard/weWEfew.png](/img/how-to-use-google-developer-tools-parsing-dcard/weWEfew.png)


### 延伸參考，如何開發 Browser Extension:


[自己的瀏覽器自己改 (Part I) －利用Firefox Addon SDK開發簡單的附加元件](http://tech.mozilla.com.tw/posts/5832/%E8%87%AA%E5%B7%B1%E7%9A%84%E7%80%8F%E8%A6%BD%E5%99%A8%E8%87%AA%E5%B7%B1%E6%94%B9-part-i-%E5%88%A9%E7%94%A8firefox-addon-sdk%E9%96%8B%E7%99%BC%E7%B0%A1%E5%96%AE%E7%9A%84%E9%99%84%E5%8A%A0%E5%85%83)

[How QCLean Works](https://speakerdeck.com/qcl/how-qclean-works-introduction-to-browser-extensions)