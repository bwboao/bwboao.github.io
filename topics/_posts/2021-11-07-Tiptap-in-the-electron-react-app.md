---
title: Adding editor into the noting app
tags: react, electron, tiptap
---    

在基本功能都會運作之後，覺得加入tiptap當作編輯器


## TLDR;
原本的文字框可以，打字，他就是很大的`textarea` tag，所以在全部的功能理論上會動之後就加入
- 找text editor
- 嫁接tiptop進入
- tiptap & react

    
## text editor
那時候參考了滿多的




## Tiptap
    
    
    
    
    
    
    
    
    
    // opening links in an external browser (e.g. chrome) instead of using electron
    // from https://stackoverflow.com/questions/32402327/how-can-i-force-external-links-from-browser-window-to-open-in-a-default-browser
    win.webContents.on('new-window', function(e, url){
        e.preventDefault();
        require('electron').shell.openExternal(url);
    })