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


看完架構其實就可以依照google dev的[Get Started](https://developer.chrome.com/docs/extensions/mv3/getstarted/)開始操作了，下面是概念跟小記錄。

## Start

通常我們要使用擴充功能，我們就會打開[商店](https://chrome.google.com/webstore/category/extensions)進行安裝、購買功能等等，但我們就是要自己寫，不是一個上架的功能，所以是一個資料夾下面有我們需要的這些檔案。另外因為我沒有上架，所以不清楚封裝跟上架需要的東西。

要啟動一個擴充功能，打開設定>擴充功能>載入未封裝內容(Settings>Extensions>Load Unpacked)，點選你已經建立好的資料夾。*或是點右上角拼圖的圖案>管理擴充功能，然後一樣打開資料夾。* 這樣擴充功能就載入了，也開始運行了。

## Manifest

這部分基本上就是按照dev的文件填寫，然後如果打錯了會直接載入不了，回報的錯誤有點爛，不會跟你說正確的寫法是什麼(基本上就是google定義怎樣就照寫)，如果有更改Manifest，就要重新載入這個擴充功能

它沒有順序的問題，就看自己怎麼分類處理，一定要有的東西有名字、介紹、版本、文件版本，我是用V3
```json
{
  "name": "Twitch Manager",
  "description": "Build an Extension! ",
  "version": "0.2", 
  "manifest_version": 3,
```
這個也是一定有的background script，指定是哪個檔案，這個js只會在載入時啟動一次。
```json
  "background": {
    "service_worker": "background.js"
  },
```
這是會用到的權限，依照你的擴充功能會用的的權限填，如果你有使用tabs但沒有在這邊填寫，會可以載入，但是有用到的會無法執行然後噴錯誤。
```json
  "permissions": [
    "tabs",
    "storage",
    "activeTab",
    "scripting",
    "webNavigation"
  ],
```
這是指定跳出的頁面跟JS，還有這個擴充功能的icon圖案，不確定為什麼action跟icons都要定義
```json
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
```json
  "options_page": "options.html",
```
Content Script會幫你把你寫好的script外加在你指定的網域中的網頁，時間點是在打開新的tab是這個網域的時候，或是從其他網域轉到符合的網域時，也就代表如果是在相符的網域內有變動，content script不會重新植入，而是就保持運作，所以如果有哪些功能需要在重新載入、轉換網頁時運作的就需要自己再多寫判斷，如果只要在同一個網域內植入一次的script就不需要。

相符的格式可以參考[macth patterns](https://developer.chrome.com/docs/extensions/mv3/match_patterns/)，我因為想在twitch的網域用自己寫的script，但不想在`/directory/`之下執行，所以就有用上exclude_matches。
```json
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
如果擴充功能還需要用上其他媒體資料，仍然需要在Manifest上填上它，例如你想要用上圖片等等就可以在這邊貼上，但是因為這部分V2與V3不同，文件好像有錯誤，所以如果需要寫可能要再查詢。
```json
  "web_accessible_resources": [
    {
      "resources": ["add.js"],
      "matches": []
    }
  ]
}
```
大致上長這樣


## UI pages

可以參考[Desiging User interface](https://developer.chrome.com/docs/extensions/mv3/user_interface)[option pages](https://developer.chrome.com/docs/extensions/mv3/options/) [popup page](https://developer.chrome.com/docs/extensions/mv3/user_interface/#popup)

首先擴充元件要有介面跟使用者互動，大致上會用到的就是popup跟option page，popup是點擊icon時會跳出的頁面，通常會出現該擴充功能的操作選單、小視窗，或是Line就是直接跳出新的視窗供使用者用，UI基本上跟普通的HTML一樣，設計一個你需要的按鈕，或是數字可以供js顯示狀態等等，最大的不同是這個頁面預設會是最小，所以container的div之類的寬度都需要寫死不能用`width:100%`之類的寫法。Option page是擴充功能罐裡頁面中的擴充功能選項，通常擴充功能都會有一些設定供使用者調整，例如顏色、時間長度等等，這個頁面就跟普通的HTML就一樣。

這些UI pages本身都跟普通的HTML一樣，最大的差異是它使用的js，也就是下一個部分的script，強大的功能都在那了。


## Scripts

這邊我分類是只要是extensions裡面的js都算是 scripts ，主要是別的點是可以用的chrome api，下一個階段紀錄，這邊主要記下架構跟作用。

### Background scripts

Background script 是整個擴充功能的基本後台，它會在安裝這個擴充功能時啟動一次，所以如果需要每次開啟chrome都要run的script可以找，這個script有全部的權限，他可以觀測網址有沒有改變等等的網址。

### Content scripts

[content script](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)
Content scripts就是可以在特定網域插入的script，可以用的權限可以參考上面可以用(i18n,storage, runtime:connect,getManifest,getURL,id,onConnect,onMessage,sendMessage)，其他的api都不能用，需要的話就可以


### Communication between content and background scripts

## Chrome APIs

### APIs

### different permissions of using Chrome API

# 遇到的問題
manifest webaccessible resources [stackoverflow]()
在content script上不能偵測網址是否有改變，只會load在開的第一次