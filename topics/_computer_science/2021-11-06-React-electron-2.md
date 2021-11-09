---
title: React + Electron 桌面app實作(2)
---
實作上遇到的困難:`contenteditable`,`fs.existSynce is not a function`

## TLDR;

接續上一篇，依序紀錄遇到的問題。
- React + Electron 環境 part 1
- contentEditable & store things part 2
- react+electron的特殊困難 part 2
- 教學資源


## contenteditable
*maintaining your own contenteditable on React is a pain*

為了讓使用者有可以有更大的自由隨意打字，同時不破會網頁上的美觀，原生的`<input>`tag和`<textarea>`都太醜了，而且會有某些限制，例如css不能改動的style或是只能固定行數的方框，如果要更動都會需要有點多工夫的workaround或是非常多的coding設定一個物件。所以在一般的`div`tag上加上`contenteditable="true"`就是一個"更漂亮"的解法，可以無痛的修改，由瀏覽器包攬大部分的動作，例如刪除、新增字元、元素大小、換行等等，也可以輕易的設定CSS style不需要考慮其他的東西。

但方便、可以有高自由度的操作的代價就是，React不會支援裡面有更改的東西，有許多例外的狀況也都需要自己handle

### React & contenteditable

首先在React中使用conteneditable會讓React噴出warning，原因是react沒有辦法管控到 contenteditable 的元素的內部更動。
```log
Warning: A component is `contentEditable` and contains `children` managed by React. It is now your responsibility to guarantee that none of those nodes are unexpectedly modified or duplicated. This is probably not intentional.
```

所以當你知道你在幹嘛的時候，就可以把警告關掉。

```jsx
// under react's render()
return(
    <div
        type="text"
        contentEditable="true"
        suppressContentEditableWarning={true}
    >
    {your.div.content}
    </div>
)
```

### how to get the contenteditable
那要怎麼拿到contentEditalbe元素裡的內容呢?既然不能透過react拿到，就只好用DOM取得了，在需要取值的地方加上function，通常會加在`onInput`這個event上面，每次輸入的時候都取值存起來，可以給其它地方用。

```jsx
// function
handleInput(e){
    //set the state when input
    this.setState({
        // or innterText depend on your request
        inputvalue: e.target.outerText
    })
}

// render()
return(
    <div
        contentEditable="true"
        ...
        onInput={(e)=>this.handleToDoInput(e)}
    >
    {your.div.content}
    </div>
)
```
當然也可以加在`onKeyDown`這個event上面，handle每次的鍵盤輸入，後來兩種我都有用到。

值得注意的是`onChange`對conteneditable的元素沒有用，可以參考這篇討論[stackoverflow](https://stackoverflow.com/questions/22677931/react-js-onchange-event-for-contenteditable)

### change the contenteditable's context

經過上面的寫法，應該會發現設定`{your.div.content}`只會改第一次，如果直接在同一個component設定react沒辦法判斷這項值會不會更新contenteditable下的元素，所以他會認定都沒有更新。

一個接續上面做法的是將你手動設的{your.div.content}換成你得到的inputvale，往上傳到你的上一層component，之後再傳回當作props，因為react知道state改變了，所以會嘗試重新render這項Component，這樣contenteditable裡面的東西就會跟著你的打字改變。

另外一種是你將自己的key改變，因為react會根據重複items的key去判斷要不要更新以節省運算，所以如果你每次打字都更動自己的key，從true變成false之類的，但這個方法不是很好，因為react就失去可以節省資源的意義。

### restore the caret position after updating
這邊我參考了兩篇 [stackoverflow1](https://stackoverflow.com/questions/3972014/get-contenteditable-caret-position) [stackoverflow2](https://stackoverflow.com/questions/6249095/how-to-set-the-caret-cursor-position-in-a-contenteditable-element-div)

做完上一步之後，你就會發現你每打一個字就會被迫更新，你的文字指標就會一直在前面，變成打字的順序是由右往左，這時候就需要取得你之前打字的位置，之後再回來重新再focus到原本的位置。

我原本的做法是，直接不管這部分，只有在`onBlur`的時候才儲存，所以期間不管怎麼打字都不會更新這個component，都是交給瀏覽器去處理contenteditable的內容，這樣也是最方便的。但如果在打字到一半沒有onBlur，點到其他東西儲存時重新整理，或是有什麼意外，剛剛打字的東西就會消失。

後來我還是有在`onInput`的時候更新，我的做法是在跟上層的存文字溝通時加上一個focus的參數，如果有需要的話就設定為true，在元件`componentDidMount()`的時候判斷，如果是真的話就將指標設定，與設定`autoFocus="true"`的表現是一樣的，但是`autoFocus`在contenditable的元素上不管用。

```jsx
// in constructor
this.inputRef = Reat.createRef();
// in componentDidMount
 componentDidMount(){
        // if want to set to the last position,need to set Range asnd Selection
        if(this.props.focus){
            let range = document.createRange();
            let selection = document.getSelection();
            
            // set range to the current node
            range.selectNodeContents(this.inputRef.current);
            range.collapse(false);
            
            // remove current selection and set selection on the range
            selection.removeAllRanges()
            selection.addRange(range);
        }
    }
```
### 但我們打的不是英文 composition
都解決完之後發現，我們正在打中文!?，中文不管是注音、倉頡、拼音等等都跟英文不一樣，我們會先打出一串字串之後，按下Enter或是空白鍵等等。那上面的做法就會導至打出來的字重複，或是意料之外的動作。這時候就得判斷是否正在[composition](https://developer.mozilla.org/en-US/docs/Web/API/Element/compositionstart_event)。
```jsx
// function
handleCompositionStart(){
    this.composition = true;
}

handleCompositionEnd(){
    this.composition = false;
}
handleInput(e){
    // if is not compositioning then store the input
    if(!this.composition)
        //do your update things
        ...
}

// render()
return(
    <div
        contentEditable="true"
        ...
        onCompositionStart={() => this.handleCompositionStart()}
        onCompositionEnd={() => this.handleCompositionEnd()}
    >
    {this.props.inputValue}
    </div>
)
```

### Paste on
contenteditable的元素非常自由，所以當你貼上一串從網頁複製來的html，他會**毫無保留**的全部都一起塞進去，連原本的顏色、格式都一起。這大概不是你想要的，所以我需要在貼上來的時候保證他只有純文字。

雖然`document.execCommand()`在MDN上面是deprecated的指令，但因為clipboad實在太難用了...，所以我後來還是用這個過時的方法
```jsx
// function
handlePaste(e){
    e.preventDefault();
    let paste = (e.clipboardData || window.clipboardData).getData('text');
    document.execCommand('insertText',false,paste);
}
// render()
return(
    <div
        contentEditable="true"
        ...
        onPaste={this.handlePaste}
    >
    {this.props.inputValue}
    </div>
)
```

### change the style
這部分conteneditable就很和善了，想要改哪種顏色、格式、大小都可以，主要有幾個會依據內容影響樹入框大小的設定，如果想要用固定的寬度，不希望文字撐開，可以參考mdn的[`overflow-wrap`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap)和[`word-break`](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)，中文的斷句比較不會有問題，但是在很長的英文單字或是網址的時候contenteditable的寬度可能會超出你想要的範圍。

## store things
記事本當然都要把打上的事情記下來，所以要handle的東西還有儲存，我用了兩種方法，一個是開記事本的時候每個筆記分別存在檔案，todo-list就存在localStorage裡面。理論上可以都先用localStorage handle再交給本地端存檔，但因為我不熟到底開怎麼做比較好，全部筆記儲存在同一個檔案也是可以的，希望之後可以學到怎麼做比較好。

### fs 
參考: [Node.js doc fs](https://nodejs.org/api/fs.html)

存在本地端的方式是利用Node.js裡面的`fs`，另外值得注意的是，Node.js是後端，所以只能在`electron.js`這邊用這些module，不能直接在renderer process(React的部分)直接呼叫，所以會利用electron的`ipcMain`和`ipcRender`溝通，等前端傳送資料給後端。另外如果出錯用try catch會比較好看出來發生什麼事，也不會停下整個程式。

```jsx
// electron.js
const { app, BrowserWindow, ipcMain, Menu } = require('electron')
// new a file
ipcMain.on('newFile', (event, arg) =>{
    let status = "new success"
    const [filename,value] = arg
    try {
        //save the file
        fs.writeFileSync(filename,JSON.stringify(value,null,2),'utf-8')
    } catch(error){
        console.log("can't new file:",error);
        status = "new fail"
    }
    // return the status to renderer process
    event.returnValue = status;
})
// saveFile
ipcMain.on('saveFile',(event,arg) =>{
    let status = "save success"
    //save the file
    try {
        // console.log("type,value = ",type,value)
        fs.writeFileSync(filename,JSON.stringify(value,null,2),'utf-8')
    } catch(error){
        console.log("can't save file:",error);
        status = "save fail"
    }
    event.returnValue = status;
})
```

```jsx
// in react part
import { ipcRenderer } from 'electron';
...
let obj = [filename,filevalue]
ipcRenderer.sendSynve("saveFile",obj)
```

這邊會遇到一個滿特別的問題: 可以跳到[這邊]({{post.url}}#React+electorn會遇到的特殊困難)

### locallStorage
第二種儲存的方式是localStorage，也可以參考IndexedDB但我沒有用。

localStorage 很簡單，直接呼叫就可以用了。
```jsx
// new a key with a value
localStorge.setItem('toDoList',todolistValue)
// delete the key
localStorge.removeItem('toDoList')
// get the value of the key
const value = localStorge.getItem('toDoList')
```

拿到之後可以利用JSON來把它轉成Object
```jsx
const storedList = localStorge.getItem('toDoList')
const parsedList = JSON.parse(storedLsit);
```

## 小心得
這邊插入一個小心得，原先我在儲存資料的時候非常困繞，想說好多參數、不同文件要再分開存一次好麻煩。但後來領悟，在js環境下什麼東西幾乎都可以是object，善用object傳來傳去不要一個一個傳遞。可以把很多東西都包成一個Object，JSON-like的物件存進檔案或是localStorage等等，之後for迴圈提出來就可以了。

## React+elctron會遇到的特殊困難 

主要就是在設定ipcRender的部分，會發現的問題，如果是單獨使用electron就不會出現這個問題，但是混在一起用就會出現。

首先要在render process用`ipcRenderer`的話，electron.js的權限設定要更改:
```jsx
const win = new BrowserWindow({
        ...
        webPreferences:{
            ...
            nodeIntegration: true,
            contextIsolation: false,
            enableRemoteModule: true,
        },
    });
```
### fs.existssync is not a function 
參考: [github issue](https://github.com/electron/electron/issues/9920#issuecomment-478826728),[webpack config](https://webpack.js.org/configuration/)

上面設定完之後，還是會出現上面這個Error，而且整個看不懂哪裡出問題了。後來查了一下才發現是electron的module，是設計不能在前端執行的，滿合理的。但理論上在前端是可以require `ipcRenderer`等前端可以執行的。

其中有看到的一個work-around是在preload.js，原本就設計可以使用全部electron api的部分，把electron整個api掛在window下面。
```jsx
//preload.js
window.electron = require('electron');
// only ipcRenderer
window.ipcRenderer = require('electron').ipcRenderer;
```
但是這樣的缺點是，會把整個api暴露在window這個global下面，而且我這樣做還是行不通。

後來依據找到的github issue所提到的，這應該是Webpack在包裝react跟electron的設定有所不同，利用`create-react-app`再將`electron`裝進來的作法，會產生的問題，像是[electron-react-boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate)這樣的模板估計就沒問題。結論是修改webpack的Target就會好了。

```JSON
export default {
...
target: "electron-renderer"
...
}
```

因為我們run start,build等等都是`react-scripts`在執行，所以我是到module裡面找到`react-scripts/config/webpack.config.js`這個檔案加上，但我不確這是不是對的做法。
```javascript
return{
    target: 'electron-renderer'
}
```

### caret and selection
餐考: [pretagteam](https://pretagteam.com/question/why-is-my-contenteditable-caret-jumping-to-the-end-in-chrome),[mozilla](https://bugzilla.mozilla.org/show_bug.cgi?id=904846)

有時候caret會不見，有遇到過消失到最左邊，跟最右邊會消失的問題。但兩次都是調整了其他東西就突然好了，所以這個段落就放兩個有遇到這個bug的連結。


## 資源
- React lifecycle
- MDN range selection
- webpack configuration
- Node.js localStorage