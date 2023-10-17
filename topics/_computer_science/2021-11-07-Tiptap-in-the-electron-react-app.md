---
title: Adding editor into the noting app
tags: react, electron, tiptap
---    

在基本功能都會運作之後，加入tiptap當作WYSIWYG編輯器


## TLDR;
原本的文字框可以，打字，他就是很大的`textarea` tag，所以在全部的功能理論上會動之後就加入
- 找text editor
- 嫁接tiptop進入
- tiptap & react

    
## text editor
編輯器是可以自己寫的，但是因為太造輪子了，所以先參考了一些其他人寫的記事本用哪咧，當時有先找最熟悉的hackMD開發的開源[CodiMD](https://github.com/hackmdio/codimd)，還有Stackoverflow自己開發的[Stacks-editor](https://stackoverflow.design/product/components/editor/)，另外直接搜尋後有找到更多例如: [froala](https://github.com/froala/wysiwyg-editor)、[quiljs](https://quilljs.com/)、[ckeditor](https://ckeditor.com/)。當時選擇TipTap純粹是最早看到有人使用在React的專案，另外CodiMD太複雜，Stackseditor是後續替換的我不太確定怎麼接到React上面，最後就用TipTap。

## TipTap

[官網](https://tiptap.dev) [install-react](https://tiptap.dev/installation/react)

TipTap是包在[Prosemirror](https://prosemirror.net/)的WYSIWYG editor，除了他接再React上面很方便之外，我也是第一次用沒什麼歷史好介紹的。

TipTap的本體很簡單，他就是一個contenteditable的方塊，沒有任何東西，如果需要多加的任何功能，就要載入extension，畫面中的按鈕等等也都要自己寫，算是比較麻煩的點，不過官網有滿多種類的examples可以參考。TipTap提供了一個方便的extension Starter-kit，包了最基本的功能，但互動的按鈕等等就要在自己寫。

值得一提的是，它現在是v2，有些舊的文件是v1，格式差距滿大的。還有開發者用的是vue.js和typescript，如果不是用這些可能需要適應一下。

### Installation

跟[文件](https://tiptap.dev/installation/react)說明的一樣簡單，不過先`npm i`需要的packages，然後將TipTap export成一個component
```jsx
// from doc
import { useEditor, EditorContent } from '@tiptap/react'
import StarterKit from '@tiptap/starter-kit'

const Tiptap = () => {
  const editor = useEditor({
    extensions: [
      StarterKit,
    ],
    content: '<p>Hello World!</p>',
  })

  return (
    <EditorContent editor={editor} />
  )
}

export default Tiptap
```

之後就將Tiptap當作普通的Component在React的專案中使用，StarterKit包含了最基本的打字、簡單的Markdown例如:`#`當作heading，codeblock等功能，

### Working with React

回傳資料也滿簡單的，editor可以選擇回傳純文字或是JSON，吃資料的時候同樣也可以選擇純文字或是JSON，JSON可以儲存文字的大小顏色等等，所以完整的儲存現在的文件就是用JSON。

```jsx
const text = editor.getText()
const content = editor.getJSON()
```


值得注意的是editor可以設定onUpdate，並不需要手動在component下面新增。如下

```jsx
let value = ""

const Tiptap = () => {
  const editor = useEditor({
    extensions: [
      StarterKit,
    ],
    content: props.content,
    onUpdate(){
      // get the text
      value = this.getText()
    },
    ...
  })

  return (
    <EditorContent editor={editor} />
  )
}
```

### Extensions

參考:

不過要有更多的控制，例如插入字串等等，可以用editor

```jsx
editor.focus()
editor.insertText("hello")
```

那這些操作通常會包在Extension裡面，Extension是Tiptap設定為可以加入editor之下的，例如預設加入的StartKit就是包含了Text,Bold等基本的extension，讓你可以進行打字(對這個是extension，沒有Text就不能打字)粗體、斜體等等的操作。

如果你想要按下f1時把所在的那行文字變成黃色的，或是想要可以插入youtube影片在裡面播放，都可以寫成Extension。通常會自訂extension，將加入

#### Keyboard shortcuts

網址:

Tiptap可以handle你在按下某些按鍵的時候，要如何動作，但他是使用Prosemirror的API，所以也是利用Extension的方式，在裡面加上`addKeyboardShortcuts()`。下面是我寫的，將Tab當作對象處理，防止從輸入中的editor切換到下一個元素。

```jsx
const CustomExtension = Extension.create({
  addKeyboardShortcuts(){
    return{
      'Tab': () =>{
        // if it is listItem pass and let the listItem extension handle rest
        if(this.editor.can().sinkListItem('listItem')){
                return false;
        }
        // else insert \t and return true to prevent any futher action triggered
        this.editor
            .chain()
            .insertContent('\t')
            .focus()
            .run()
        return true;
      }
    }
  }
})
```


#### external links

#### CodeBlockLowlight

A code languae highlighting extension, thought  I would be as easy as before but not.

The extension is duplicated with the StarterKit extension

```jsx
// configure the starerkit, remove the duplicated CodeBlock extension
...
extensions: [{
  StarterKit.configure({
    CodeBlock: false,
})
...
```
The documentation didn’t explain, but need to configure to run the highlight function 
```jsx
CodeBlockLowlight.configure({
            lowlight,
})
```
What’s lowlight?

```jsx
// I suppose this is from the extension, don't know how it works
// This is part of highlight.js 
import lowlight from 'lowlight'
```
Manual set the CSS. The document did give an example, so I just copied it.


### Click on external links

因為我在原先的設計上是不會用到任何外部的資源(網路上)，所以在權限上設定了很寬鬆的限制

```jsx
// opening links in an external browser (e.g. chrome) instead of using electron
// from https://stackoverflow.com/questions/32402327/how-can-i-force-external-links-from-browser-window-to-open-in-a-default-browser
win.webContents.on('new-window', function(e, url){
    e.preventDefault();
    require('electron').shell.openExternal(url);
})
```