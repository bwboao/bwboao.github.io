---
title: chrome extension 初試
---

這次主要是紀錄一下寫chrome extension，這次要寫的時候發現好像改版變成V3了，發現連doc有些地方都會寫錯，不過大部分的內容文件沒錯。

如果要寫的話，最基本的就是要會JS跟HTML/CSS等網頁基礎，所以如果不會這些的話請從基礎開始。


官方文件：[google dev extensions](https://developer.chrome.com/docs/extensions/)

# Chrome Extension
chrome extension是包在chrome底下可以幫你執行code的擴充功能，基本上就是chrome會在後面幫你執行的js跟會彈出的html，並且可以操作瀏覽器本身，例如關閉視窗、彈出視窗等等。
跟普通的js不同的是，提供了chrome API可以用，除了用來針對瀏覽器本身的操作、分頁的網址是否有改動，還可以儲存資料在chrome.storage等瀏覽器提供的位置，但相對的，儲存的東西就需要檢查安全性，所以有些操作是不能在某些區域執行的。

# Architecture 架構
可以參考： [Architecture overview](https://developer.chrome.com/docs/extensions/mv3/architecture-overview/), [Extension development overview](https://developer.chrome.com/docs/extensions/mv3/devguide/)

在開始看google文件中的"Getting started"章節前，最好其實可以先看看架構，尤其對第一次寫這類東西的我，直接跟著開始寫真的超一頭霧水，不如先理解extension是如何運作的。


擴充功能的架構分別是
- Manifest 
- Scripts
	-	background scripts
	-	Content scripts
- UI elements
- Option Pages

Manifest是一個紀錄版本、擴充功能名稱、作者、使用的權限、媒體檔案等等的JSON文件，用來跟chrome溝通這個擴充功能用了哪些東西，要吃哪些檔案，是最基本的也是第一個創建的檔案。接下來是scripts，有分成掛在網頁下執行js如同打開console在dev tool下面執行js一樣的Content Script，跟在chrome的後台執行、有更多權限的Background Script。除了腳本，當然會有視窗跟使用者溝通，最簡單的就是右上角出現icon點下去會出現的popup小視窗，它其實就是基本的HTML+js，使用者可以用這個部分設定、點下去改變網頁等等。最後就是設定頁面，也是基本的HTML+js，但是在你點下設定這個擴充功能才會出現的頁面。


看完架構其實就可以依照google dev的[Get Started](https://developer.chrome.com/docs/extensions/mv3/getstarted/)開始操作了，下面是get started沒有提到的概念跟小記錄。

# Start

通常我們要使用擴充功能，我們就會打開[商店](https://chrome.google.com/webstore/category/extensions)進行安裝、購買功能等等，但我們就是要自己寫，不是一個上架的功能，所以是一個資料夾下面有我們需要的這些檔案。另外因為我沒有上架，所以不清楚封裝跟上架需要的東西。

要啟動一個擴充功能，打開設定>擴充功能>載入未封裝內容(Settings>Extensions>Load Unpacked)，點選你已經建立好的資料夾。*或是點右上角拼圖的圖案>管理擴充功能，然後一樣打開資料夾。* 這樣擴充功能就載入了，也開始運行了。

# Manifest

這部分基本上就是按照dev的文件填寫，然後如果打錯了會直接載入不了，回報的錯誤有點爛，不會跟你說正確的寫法是什麼(基本上就是google定義怎樣就照寫)，如果有更改Manifest，就要重新載入這個擴充功能

它沒有順序的問題，就看自己怎麼分類處理，一定要有的東西有名字、介紹、版本、文件版本，我是用V3
```
{
  "name": "Twitch Manager",
  "description": "Build an Extension! ",
  "version": "0.2", 
  "manifest_version": 3,
```
這個也是一定有的background script，指定是哪個檔案，這個js只會在載入時啟動一次。
```
  "background": {
    "service_worker": "background.js"
  },
```
這是會用到的權限，依照你的擴充功能會用的的權限填，如果你有使用tabs但沒有在這邊填寫，會可以載入，但是有用到的會無法執行然後噴錯誤。
```
  "permissions": [
    "tabs",
    "storage",
    "activeTab",
    "scripting",
    "webNavigation"
  ],
```
這是指定跳出的頁面跟JS，還有這個擴充功能的icon圖案，不確定為什麼action跟icons都要定義
```
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "/images/TWITCH16.png",
      "32": "/images/TWITCH32.png",
      "48": "/images/TWITCH48.png",
      "128": "/images/TWITCH128.png"
    }
  },
  "icons": {
    "16": "/images/TWITCH16.png",
    "32": "/images/TWITCH32.png",
    "48": "/images/TWITCH48.png",
    "128": "/images/TWITCH128.png"
  },
```
這是設定頁面是使用哪個檔案，他也可以有外加的JS檔案
```
  "options_page": "options.html",
```
Content Script會幫你把你寫好的script外加在你指定的網域中的網頁，時間點是在打開新的tab是這個網域的時候，或是從其他網域轉到符合的網域時，也就代表如果是在相符的網域內有變動，content script不會重新植入，而是就保持運作，所以如果有哪些功能需要在重新載入、轉換網頁時運作的就需要自己再多寫判斷，如果只要在同一個網域內植入一次的script就不需要。
```
  "content_scripts": [
    {
      "matches": [
        "https://www.twitch.tv/*"
      ],
      "exclude_matchs": [
        "https://www.twitch.tv/directory/*",
        "https://www.twitch.tv/settings/*"
      ],
      "js": [
        "content.js"
      ]
    }
  ],
```

```
  "web_accessible_resources": [
    {
      "resources": ["add.js"],
      "matches": []
    }
  ]
}
```
大致上長這樣
```
{
  "name": "Twitch Manager",
  "description": "Build an Extension!",
  "version": "0.2",
  "manifest_version": 3,
  "background": {
    "service_worker": "background.js"
  },
  "permissions": [
    "tabs",
    "storage",
    "activeTab",
    "scripting",
    "webNavigation"
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "/images/TWITCH16.png",
      "32": "/images/TWITCH32.png",
      "48": "/images/TWITCH48.png",
      "128": "/images/TWITCH128.png"
    }
  },
  "icons": {
    "16": "/images/TWITCH16.png",
    "32": "/images/TWITCH32.png",
    "48": "/images/TWITCH48.png",
    "128": "/images/TWITCH128.png"
  },
  "options_page": "options.html",
  "content_scripts": [
    {
      "matches": [
        "https://www.twitch.tv/*"
      ],
      "exclude_matchs": [
        "https://www.twitch.tv/directory/*",
        "https://www.twitch.tv/settings/*"
      ],
      "js": [
        "content.js"
      ]
    }
  ],
  "web_accessible_resources": [
    {
      "resources": ["vertical.js"],
      "matches": []
    }
  ]
}
```

# UI pages

option pages popup page

# Scripts


### Content scripts
[content script](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)

### Background scripts

### Communication between content and background scripts

# Chrome APIs

## APIs

## different permissions of using Chrome API

# 遇到的問題
manifest webaccessible resources [stackoverflow]()
在content script上不能偵測網址是否有改變，只會load在開的第一次