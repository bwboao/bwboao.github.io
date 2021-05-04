---
title: Chrome extension 初試
---

這次主要是紀錄一下寫chrome extension，這次要寫的時候發現好像改版變成V3了，發現連doc有些地方都會寫錯，不過大部分的內容文件沒錯。

如果要寫的話，最基本的就是要會JS跟HTML/CSS等網頁基礎，所以如果不會這些的話請從基礎開始。


官方文件：[google dev extensions](https://developer.chrome.com/docs/extensions/)

## Chrome Extension
chrome extension是包在chrome底下可以幫你執行code的擴充功能，基本上就是chrome會在後面幫你執行的js跟會彈出的html，並且可以操作瀏覽器本身，例如關閉視窗、彈出視窗等等。
跟普通的js不同的是，提供了chrome API可以用，除了用來針對瀏覽器本身的操作、分頁的網址是否有改動，還可以儲存資料在chrome.storage等瀏覽器提供的位置，但相對的，儲存的東西就需要檢查安全性，所以有些操作是不能在某些區域執行的。

## Architecture 架構
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

```markdown
Chrome extension scripts
├─────────────── Background script 
│                   ├─(Message)
│                   │
├─[matched reg]── Content script 
│                   ├─(Message)
│                   │
└─────────────────Element script(e.g. `popup.js`)─── UI Element
```


### Background scripts

Background script 是整個擴充功能的基本後台，它會在開啟瀏覽器時啟動一次，所以如果需要每次開啟tab都要run的script可以找看看有沒有這個api (`chrome.runtime`) 或是掛在content script下。這個script有全部的權限，他可以用觀測網址有沒有改變等等的網址，或是開新視窗等等的chrome操作，所以這背景的script有可能惡意使用權限。

### Content scripts

參考： [content script](https://developer.chrome.com/docs/extensions/mv3/content_scripts/)


Content scripts就是可以在特定網域插入的script，可以用的權限可以參考上面可以用(i18n,storage, runtime:connect,getManifest,getURL,id,onConnect,onMessage,sendMessage)，其他的api都不能用，需要的話就要跟background scripts溝通，用`chrome.runtime.onMessage`傳訊息給後台。

Content script最直觀，也最直接的功用就是調整網頁的畫面，例如我需要在Google doc跳出一個框框通知我中午十二點了，該吃飯了；或是自懂幫我按下某個按鈕。

以下是我參考其他人寫的，我想辦法觀測twitch的聊天點數按鈕出現就自動按下領獎按鈕，那這個content script當然就是當網址符合`https://twitch.tv`才植入。這個funtion的作用是觀測如果這個element之下有任何改變就想辦法判斷可不可以點，所以在網址更改等等之後就會呼叫這個函數，在重新對我需要的element加上observer。

```js
function addMutation(){
  // Check if in the page
  if (document.contains(document.getElementsByClassName('community-points-summary')[0])){
    // select the node that will be observed
    const targetNode = document.getElementsByClassName('community-points-summary')[0];
    // Create an observer instance
    observer = new MutationObserver(function(mutations){
      mutations.forEach(function(mutation){
        //if the mutation is add nodes , and the tag is not <span> (I need button)
        if (mutation.addedNodes.length > 0 && mutation.target.tagName !== 'SPAN'){
          console.log('Detect Nodes added and not SPAN');
          // get all buttons inside 'community-points-summary'
          var buttons = document.querySelector('.community-points-summary').querySelectorAll('button');

          //Click each button, except for the first 
          // for Each: https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
          buttons.forEach( function (currentButton, index, arr) {
            if (index != 0){
              //click the button and show the button clicked
              console.log(currentButton);
              currentButton.click();
              //this function messages the background script to total the times clicked
              reportClicked();
            }
          }); 
        }
      })
    })
    //Options for the observers
    const config = { attributes: true, childList: true, subtree: true};

    // pass in the target Node , and the config
    observer.observe(targetNode,config);
  }

}
```

### Communication between scripts

上面有提到如果有需要的api前面後面權限不同所以需要用`chrome.runtime.onMessage`傳訊息溝通，這邊記下我寫的功能，我需要的東西是我觀察到網址改了，那我就通知前面的content script要重新抓到新的頁面的element。

在打開一個chrome tab的content script時第一次會抓，但是如果換成同樣都是符合content script match patterns的網址，它會沿用已經插入的script，所以不會再重新執行一次整個script。流程如下

```
open a new website --> matched -->
content script will fire(inject the fitst time) --> 
change url --> matched -->
content script will not fire (use the injected one already)
```

所以需要在後台監測有改網址後，發訊息給插入的content script。如果前面的網頁沒有符合，那就不會插入content script，也就不會接收到訊息。Content script接到訊息之後，就可以執行相對應的動作

```
change url ──> background script send message ───────┐
 └─────> macthed ──> content script(already running) ┴─> receive + action
```

下面是根據上面描述的狀況我寫的，權限的部分，我有使用`chrome.tabs`，所以要在manifest那邊加上。

在background script這邊加上如果有觀測到網址改變的Listener，觀測到後傳送類似JSON的object給前面的content script。
```js
chrome.webNavigation.onHistoryStateUpdated.addListener( (details)=>{
  // details is the object containing info of the urlchanging event
  chrome.tabs.sendMessage(details.tabId, { // this part is self-defined JSON format
    greeting: "pageurlchange", // this is self defined
    url: details.url, //detail has attribute url
    details: details //
  }, function(response){
    console.log(response.farewell)
  })
});
```

在 Content script 的部分加上Listener，首先判斷傳來的訊息是否是我要的，可能還有其他狀況後台也會傳過來。判斷是`"pageurlchange"`之後，我就會執行`main()`重新抓取更換網址後的頁面的資料。
```js
chrome.runtime.onMessage.addListener(
  function (request, sender, sendResponse) {
    //onMessage.addListener uses function as callback, the args are fixed
    //reset the Mutation and do everything
    console.log(request)
    if (request.greeting == "pageurlchange") {
      // send the request's response back
      sendResponse({farewell: "gotit"})
      // do everythiing again
      main();
    }
  }
);
```

## Chrome APIs

讓擴充功能的script跟普通的javascript不同的是它可以使用chrome的api，針對瀏覽器本身進行操作，或是其他的操作，例如上方提到的content script跟background script的message就是chrome api其中一個，利用chrome提供的功能才造就各種強大的擴充功能。

### APIs

這邊稍微記下我有用的api，詳細的還是請參考[api reference](https://developer.chrome.com/docs/extensions/reference/)

因為我寫的功能很少，也不太需要溝通協調不同的功能所以沒有用到`i18n`等等其實滿基本的東西。

`chrome.runtime`:
正在運作時可以用的api，例如後台用的`chrome.runtime.onMessage`就是等待有任何人傳訊息的api。

`chrome.tabs`:
跟分頁有關的api，我主要用的是`chrome.tabs.sendMessage`，`tabs`會送給後台訊息


`chrome.storage`:
依照文件說明的，`chrome.storage`可以儲存資料在瀏覽器，要注意的是這邊不能儲存機密資料，分為`chrome.storage.sync`是可以同步所有同樣帳號的chrome都可以儲存、提取的資料和沒有sync的就是只有儲存在本地。

通常我們會在第一次安裝的時候設定這個資料，格式是類似JSON，用`set()`指定值
```js
chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.sync.set({ 'clickCount': 0});
};
```
對這個資料進行操作，我的做法是先用`.get()`把這個資料取出來，做加法等等的操作後再指定給他用`.set()`儲存
```js
// default to 0  if storage has no clickCount (i supposed it behaves like this)
chrome.storage.sync.get({clickCount: 0}, (results) => {
  chrome.storage.sync.set({'clickCount': results.clickCount+1 })
})
```

如果要印出這個值得話可以在`console.log`直接用
```js
console.log('Default count is set to storage clickCount, `clicks: ${clickCount}`');
```


或是可以搭配popup.html+js
```xml
<!-- popup.html -->
<div id="showClickNumberDiv">
  <span>Twitch Manager has clicked: </span>
  <span id="showClickNumber"></span>
  <span>times</span>
</div>
```
```js
//popup.js
let showClickNumber = document.getElementById("showClickNumber");
// initailize the button with the count storaged
chrome.storage.sync.get("clickCount",({clickCount}) => {
  showClickNumber.innerHTML = clickCount;
})
```
每次點icon都會重新打開popup.html和js，所以每次都會重新抓數值，不需要動態的去抓`chrome.storage`裡面的資料是否有改變。


### different permissions of using Chrome API

就上面有提到的，content script api的權限跟background script不一樣，在content script上面執行沒有權限的動作不會有錯誤（至少我嘗試的時候是這樣），所以在寫的時候要檢查這個地方有沒有具有權限

## 遇到的問題
1. manifest webaccessible resources [stackoverflow]()
2. 在content script上不能偵測網址是否有改變，只會load在開的第一次
3. 用content script執行`chrome.tabs`