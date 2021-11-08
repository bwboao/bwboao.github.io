---
title: React + Electron 桌面app實作(1)
---

如何架起react+electron環境

## TLDR;

很醜的code: [github](https://github.com/bwboao/noting_app)

只記錄大致上的東西，遇到最多的困難都是React要怎麼規劃component跟如何傳遞資料
- React + Electron 環境 part1
- contentEditable
- 遇到的困難
- 教學資源

### 起源

這個project本來是為了練習如何寫React, 剛好看到了electron這個框架，可以把javascript包成桌面app，所以就決定寫一個"小記事本"。原本想到的功能有筆記、日記，後來加上to-do list，沒想到看似簡單的這些功能卻真的一堆問題，學了滿多。為了單純練習我決定不用其它套件，先寫個會動的版本，這篇記錄的是沒有用仼何其它套件的部分。

如果要看React+electron的架設可以直接從[React+electron的環境]({{post.url}}#環境)看起來


## React

[官方文件](https://reactjs.org/docs/getting-started.html)

React是現在很流行的library，也可以當是一個框架(我沒有很了解其中的差異)。將東西都當成component(元件)，可以重複利用，設定component的state(狀態)等等。官方的文件寫的滿好的，如果是新手建議從文件看起，

### 開始
因為我也是新手，所以是參考最簡單的方法，這邊可以看下方的環境架設，因為是一起做的。另外我這次都是用[`npm`](https://docs.npmjs.com/cli/v7/commands/npm)來管理套件，如果是用`yarn`的，抱歉了。

```sh
# use create-react-app to create a directory
# npx will install react and its dependencies
npx create-react-app noting-app
```
這樣基本的東西就都有了，你就可以開始寫React了，如果是純粹的React，之後要deploy到網路上，用以下就可以打開server，開始邊寫看邊render後的樣子.

```sh
npm run start
```
我就不介紹React的各種特性了，文件很清楚，也有更多其它資源可以參考。
- Component
- State
- Life Cycle
- Hooks

### component 構想

基本上規劃Component如何接起來，對我來說才是最困難的，React的生命週期都是由上往下，上面更動了什麼可以往下傳，如果有更動React就會管理要不要重新render。

我的想法是todo list要一直在畫面上，可以切換要不要Note Journal，我一開始先用Adobe XD劃出我想像中需要的功能跟按鈕，最後得出我應該可以這樣大致分出幾個Component。

```
<App/>
│ ├─ <Note/> ─────<Editor/>
│ └─ <Journal/>───<Editor/>
└─── <To-do List/>
```

## electron 
[官網](https://www.electronjs.org/)

electron是可以將 HTML, Javascript, CSS包成桌面app的套件，不一定要跟React串在一起，仼何前端library應該都可以。我這次串再一起純粹是為了練習，electron強大的地方是它基本上就是後端，可以利用Node.js將東西儲存起來等等，也可以純粹將在網路上運作的網址包成桌面app，不過那樣就得自己面對攻擊，所以我寫的純粹都是在本地端運行，盡量不要用到有任何載入網路上的資源的部分。

### electron 簡介
[quick start](https://www.electronjs.org/docs/latest/tutorial/quick-start)

electron的官網文件比較混亂一點，雖然在我寫的期間，他有大更新他的說明文件，沒有很好懂但讀quick start大致上就可以理解electron是怎麼運作的。

基本上electron就是將你寫的網頁包在瀏覽器當作一個app，分會有後端的部分，也是主要的服務(main process)，拿來控制你打開的視窗、瀏覽器，有很多的electron指令只能在這個範圍使用(為了安全，也合理因為大部分是後端資料處理)。前端的就是被瀏覽器執行的網頁，而electron也有提供方法讓前後可以溝通，可以將前端得到的資料儲存，或是使用者操作打開瀏覽器、開新的視窗等等。如果你寫好了，就可以打包成桌面app，例如Discord就是有名的使用electron的程式，使用DC時，有許多時候可以看出他基本上就是網頁。

### 前後端溝通
參考: [doc](https://www.electronjs.org/docs/latest/api/ipc-main)

electron可以透過`ipcMain`在main process等待，`ipcRenderer`在前端任何地方等待傳遞，這部分跟之前寫chrome extension滿像的，下面是官網的範例。

```javascript
// In main process
const { ipcMain } = require('electron')
ipcMain.on('asynchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.reply('asynchronous-reply', 'pong')
})

ipcMain.on('synchronous-message', (event, arg) => {
  console.log(arg) // prints "ping"
  event.returnValue = 'pong'
})
```
```javascript
// In renderer process (web page).
// NB. Electron APIs are only accessible from preload, unless contextIsolation is disabled.
// See https://www.electronjs.org/docs/tutorial/process-model#preload-scripts for more details.
const { ipcRenderer } = require('electron')
console.log(ipcRenderer.sendSync('synchronous-message', 'ping')) // prints "pong"

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log(arg) // prints "pong"
})
ipcRenderer.send('asynchronous-message', 'ping')
```
另外值得一提的是在main process你隨時都可以利用`event.reply(...)`來傳遞訊息到前端，不一定要在`ipcMain.on()`裡面才可以回應，這對由electron建立的選單很有幫助，參考[Render process menu](https://www.electronjs.org/docs/latest/api/menu#render-process)，例如右鍵產生的選單，點選項後可以再回傳資料到前端更新頁面。

## 環境
我是參考了這篇[section文章](https://www.section.io/engineering-education/desktop-application-with-react/)

install the npm libraries
```shell
#basic core of this project
npx create-react-app noting_app
cd noting_app
npm install --save-dev electon
#extra for setting up the dev environment
# i = install , -D = --save-dev
npm i -D electron-is-dev
npm i -D concurrently wait-on
```

my directory is set like this
```
....
├─ public/
│   ├──electron.js
│   └──preload.js
├─ src/
│   └─(React component js or folders...)
├─ App.js
...
```

在package.json 加上main的欄位，讓webpack知道是哪個檔案在執行electron
```JSON
"main": "./public/electron.js"
```

我建立的`electron.js`就是electorn官網quick start提到的`main.js`也就是electorn的main process。electron如何控制這個App開關視窗，後端的操作Node.js等等都在這裡，當然可以再從其他js檔案import近來。以下都是最簡單的設定加上為了讓開發更方便的，開發的時候我們想要react先跑起來，electron再聽react跑起來的網頁。而不是開發的時候就應該載入本地端的檔案(已經build過的react)。
```javascript
const { app, BrowserWindow, ipcMain, Menu } = require('electron')
const isDev = require('electron-is-dev')
const win = new BrowserWindow({
        width: 1200,
        height: 670,
        // backgroundcolor when nothing is loaded
        backgroundColor: '#19243B',
        webPreferences:{
            preload: path.join(__dirname,'preload.js'),
            nodeIntegration: true,
        },
    });
    // If is dev then load the localhost(react), else open the build file
    win.loadURL(
        isDev
        ? 'http://localhost:3000'
        : `file://${path.join(__dirname, '../build/index.html')}`
    );
    // Open the Devtools.
    if(isDev){
        win.webContents.openDevTools({ mode: 'detach'});
    }

}

app.whenReady().then(()=>{
    createWindow()
    app.on('activate', function() {
        if (BrowserWindow.getAllWindows().length === 0) createWindow()
    })
})
// close app when all windows are closed
app.on('window-all-closed', function(){
    if(process.platform !== 'darwin') app.quit()
})
```

App.js是下`create-react-app`之後自動生成的，你可以依據你要寫的app把內容都改掉
```javascript
class App extends React.Component{
    render(){
        // My own Components...
        return(
            <div  className="app">
                <MenuBar/>
                <Content/>
                <ToDoList/>
            </div>
        )
    }
}
```
最後我們要讓可以下一個指令就讓程式都跑起來，所以修改package.json裡面的scripts欄位，react基本上已經幫你加上`start`,`build`,`test`,`eject`這幾個基本指令了，用途可以參考文件。這邊我們加上`dev`和`electron`方便我們跑所有的東西跟單獨測試electron是否有問題。package.json內容如下，當然這些都還可以自己修改。

```JSON
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "dev": "concurrently -k \"npm start\" \"npm:electron\"",
    "electron": "electron .",
  },
```

之後下npm指令就可以開始開發了，注意的是React的部分，會實時更新，你有存檔server就會更新網頁，但是如果是electron的部分，就需要關掉electron再重新下一次指令。

```sh
npm run dev
```

## Electron-builder
參考: [medium文章](https://medium.com/@johndyer24/building-a-production-electron-create-react-app-application-with-shared-code-using-electron-builder-c1f70f0e2649 ) [官網](https://www.electron.build/)

寫完之後當然要打包成exe，或是dmg等可以安裝成桌面app的檔案啦，這就是當初選擇electron的原因啦，不然直接寫一個網頁就好了，我用的是`electron-builder`，electron官網還有推薦用`electron-forge`但我沒用過。

主要有兩個部分，因為我這邊包起來的還滿簡單的，環境設定也跟上面參考的文章不太一樣，我只需要先下載`electron-builder`，新增npm指令和build欄位就可以了。

在package.json加上`package`欄位如下:
```JSON
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    ...
    "package": "electron-builder build --win -c.extraMetadata.main=build/electron.js --publish never"
  },
 ...
"build": {
    "files": [
      "build/**/*",
      "node_modules/**/*"
    ],
    "icon": "./src/icon.icn"
  },
```

其中`-c.extraMetaData.main=build/electorn.js`這個參數是跟electron-builder說明要使用的electron main file是哪個，在package.json中我們是指定在`public/electron.js`但是在`npm build`之後會被打包在build資料夾下面，跟package.json中指定的衝突了，如果使用public裡面的檔案所有使用相對位置的資料都會壞掉，所以就特別指定覆蓋package.json中的欄位。

`--publish never`這個參數，是代表是否要在之後發布新的版本時更新，如果有設定的話，程式打開時會到特定的發布位置確認是否有新的版本，但因為我們沒有想要這樣更新所以就下never，如果是正式的軟體，通常會設定需要更新就向例行的chrome更新等等。

如果是在mac上的話可以改成，
```sh
electron-builder build --mac -c.extraMetadata.main=build/electron.js --publish never
```
或是只想要portable的exe而不是安裝檔的話
```sh
electron-builder build --win portable -c.extraMetadata.main=build/electron.js --publish never
```

更多的選項請參考官網的[cli](https://www.electron.build/cli)和[configuration](https://www.electron.build/configuration/configuration)

注意這邊build欄位所加上的files是指要包起來的檔案，而我們經過react build之後的檔案都在build/資料夾裡面，當然還要記得包含你有使用的node_modules。實務上打包的時候都會刪掉不需要的modules讓打包完的大小更小(官網也有提到)，但是我不知道怎麼做qwq，可以之後多深入研究。

之後下以下的指令就可以打包成你要的exe了，`electorn-builder`會把所有的檔案丟到dist資料夾裡面
```shell
# build react into a static resource
npm run build
# package the built static files
npm run package
```


### electron-builder on Windows

我在用`electron-builder`時遇到了一些問題，但當時狀況有點混亂，我同時改了很多東西，就大致紀錄一下

#### 1. permission: windows defender
可能問題:[github issue](https://github.com/electron-userland/electron-builder/issues/5759)
我可能在build的欄位，或是哪邊沒有設定對檔案名，雖然我後來都刪掉了發現也可以跑，但第一次跑的時候被Windows Defender認定是病毒，所以在跑出來的時候就會被刪掉，`electron-builder`也會跟你說permission denied。

#### 2. permission denied again
之後某次build的時候，它還是跟我說是permission denied，但理論上我已經修正了問題。後來我發現工作管理員中已經有複數個electron正在執行，可能是前幾次build成功後我打開的electron app是寫壞的，它沒有出現任何視窗，但是仍在背景執行中，所以停掉所有process之後就可以再次打包了。

#### 3. module not found
可能的問題: [stackoverflow](https://stackoverflow.com/questions/35682131/electron-packager-cannot-find-module)
這段是我有點不明所以的部分，因為是我幾次都pacakge成功之後，突然一次它噴出我有某個module not found。注意不是在打包成功後出現，而是在打包時出現的，這好像是兩種問題。

後來我是全部都重新乖乖下錯誤說有缺的module
```sh
npm intall --save <module_name>
```
但因為我那時後重新安裝整個`npm module`資料夾，還安裝`yarn`等等，所以我不確定是哪個方法修正了這個問題。


還有遇到的問題，接下一篇part2好了。

## 資源
- React [官網docs](https://reactjs.org/docs/)
- electron [官網](https://www.electronjs.org/)
- electron-builder [官網](https://www.electron.build/)
- 善用google，和github上面的issue有時候會遇到有人有一樣的問題。