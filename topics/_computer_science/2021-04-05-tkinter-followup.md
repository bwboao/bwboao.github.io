---
title: Tkinter 補充
---

上篇紀錄了大致的Tkinter要怎麼做，沒想到拯救了過一年之後再改同一個專案的我

## TDLR;

這篇主要紀錄這次用到的東西
- Messagebox + TopLevel
- Multiprocessing + Multithreading

可以參考:[Tkdocs windows](https://tkdocs.com/tutorial/windows.html)

上一篇本來要整理Theme，但是最大的tk說明網站[effbot.org](https://effbot.org/tkinterbook/)的tkinterbook，不知名原因收掉了，反而官方的文件比他還不全，有點微妙...。我後來也沒有用到theme的部分，它大致上是可以改變tk整體的樣式，例如：外框、列表、按鈕的造型，但後來沒有需要就沒使用到。

## Popping up another window

我們常常需要多開另一個視窗，例如跳出選擇檔案的視窗、選單，或是一個警告、訊息（「你是否要關閉？關閉、取消」lol），或是進度條等等的視窗，這時候我們就會用到這些。大致上分為訊息公告類的預設物件，跟直接開啟一個新視窗，可以跟主app一樣自訂各種東西功能，一路加上去，但就是要處理關閉等待的一些觸發。

### Default windows
有許多跟作業系統相關的操作視窗，在tkinter內都已經幫你包好了，不需要在用`os`或是`pywin32`等等的module，自己跟作業系統溝通。連回傳之後的存擋也都會幫你處理好。


以下的範例參考自:[Tkdocs](https://tkdocs.com/tutorial/windows.html#dialogs)


1. 選擇檔案
```py
from tkinter import filedialog
filename = filedialog.askopenfilename() #open file
filename = filedialog.asksaveasfilename(defaultextension=".jpg") #save as jpg
dirname = filedialog.askdirectory()
```
2. 選擇顏色
```py
from tkinter import colorchooser
colorchooser.askcolor(initialcolor='#ff0000')
```
3.	選擇字體
```py
l = ttk.Label(root, text="Hello World", font="helvetica 24")
l.grid(padx=10, pady=10)
# define a function for binding
def font_changed(font):
    l['font'] = font
# the use of font chooser has not been fully supported please se TKdocs for more info
root.tk.call('tk', 'fontchooser', 'configure', '-font', 'helvetica 24', '-command', root.register(font_changed))
root.tk.call('tk', 'fontchooser', 'show')
```

### messagebox
最簡單的方式就是用tkinter內建的function，`messagebox()`，它包含了幾種預設的視窗，Tkinter通常會處理得比自己寫的好，在處理完之後也會回傳選擇。

1. 警告
警告的視窗會有音效、特效，並且有ok的選項，沒有點完之前不能使用主程式。
```py
from tkinter import messagebox
messagebox.showwarning(message="This is a warning")
```
2. 提示
提示會有ok的選項，
```py
messagebox.showinfo(message="TK Info", # the big message
	detail="This is the Information", #the detail message 
	title="Information", # the title on the window box
	default="no" #default button, usually is 'ok',can be 'abort, retry, ignore, ok, cancel, yes, or no')
```
3. 詢問（是/否）
```py
messagebox.askyesno(message="Do you want to do this?",
	detail="this operation will delete files",
	default="no")
```
4. 其他
還有其他的視窗，這邊列出全部，還有它會跳出的按鈕相對應的回傳值

 - `showinfo`: 			⇒ "ok"
 - `showwarning`: 		⇒ "ok"
 - `showerror`: 		⇒ "ok"
 - `askokcancel`: 		⇒ True (on ok) or False (on cancel)
 - `askyesno`: 			⇒ True (on yes) or False (on no)
 - `askretrycancel`: 	⇒ True (on retry) or False (on cancel)
 - `askquestion`: 		⇒ "yes" or "no"
 - `askyesnocancel`: 	⇒ True (on yes), False (on no), or None (on cancel)

### TopLevel

如果要跳出一個新的自訂視窗，就要用`TopLevel`

```py
from tkinter import *
# create the main app
app = tk.Tk()
app.geometry('800x600')
app.title("Main app")
# create a button to open the new window
btn = ttk.Button(app, text="choose size", command=lambda: create_window(app))
btn.grid(row=0,column=0)

app.mainloop()
#
def create_window():
	# bind the toplevel under app
	choose_window = Toplevel(app,bg="#eee")
	choose_window.title("Choosing window")
	# let the window be centered on the screen
	choose_window.geometry('400x200+%d+%d' % (app.winfo_screenwidth()/2-250,app.winfo_screenheight()/2-50))
	# not resizeable
	choose_window.resizable(FALSE,FALSE)
	# add more widgets on it
	...
```

創建完之後，就跟主要的程式一樣，可以綁更多的物件上去，加上按鈕、顯示畫面等等的，通常我們可能會綁確認鍵，確認後會關閉這個視窗。
```py
	...
	button=ttk.Button(choose_window,text="ok I choose this",
		command=lamda: confirm_choose(choose_window))
	...
# define an funciton outside the create_window() function
def confirm_choose(choose_window):
	# do something about the choose
	# destory the window
	choose_window.destory()
```

在按下確認前，通常我們會想要主程式等待這個選擇視窗結束或按取消（像是選擇檔案、警告這類的視窗），那我們會用等待它出現、把它提到最上面、然後專注到它(像是滑鼠或按鍵已經選到)、最後請主程式等待它。

```py
	# inside the create_window()
	...
	choose_window.wait_visibility()
	choose_window.lift()
	# don't know the difference between grab_set() and focus() 
	choose_window.grab_set()
	choose_window.focus()
	choose_window.wait_window()
```

可以參考:[What does the wait_window() do?](https://stackoverflow.com/questions/28388346/what-does-thewait-window-method-do)



## Multiprocessing

`tk`在處理視窗的時候，根據我上一篇提到[對tk的運作的理解]({% link _computer_science/2020-02-18-tkinter.md%}#架構-structure)是沒有錯的，他會先處理完callback之後才會回到主要mainloop，這樣的運作方式在你要跑很冗長的動作時會卡住整個畫面，例如`progressbar()`在顯示現在運算的進度時，如果一個段落的運算很久那進度條根本就不會動，所以會用平行化的方式，使用`multithread`或是`multiprocessing`等等的module來實行。下面記下要注意的地方。

大致上遇到幾個問題
- Using Tk with multiprocessing
- Using multiprocessing on Windows

### Tk with multi-*
When using `multithread` there should be no problem, but there are sequences that we need more processing powers when multithreading just doesn't work. Using multiprocessing and multithreading is simple just call the modules `multithread` and `multiprocessing` , but you need to be careful about as [suggested](), fully pack all function especially the `mainloop()` under the `__main__`. So that multiprocessing does not mess with the main loop and pop out another `tkinter.Tk()` main window.

```py
import tkinter as tk
import multiprocessing
# lock everything inside __main__
if __name__ == '__main__':
	# creating the application main window and its name
    app = tk.Tk()
    # do things
    ...
    # do multiprocessing things
    def worker():
    	pass
    input_list = ['a','b','c'] # things to give to worker
    with multiprocessing.Pool(cpus) as pool:
    	result = pool.map_async(worker,input_list)
    # call the mainloop() to activate the app
    app.mainloop()

```

Another usual circumstances when using multiprocessing, we want to show the progress by the `progressbar` widget. The problem is when calling `update_idletasks()` or `update()` at child process doesn't update the main process(the progressbar). After a long search on Google LUL , a better approach is to open a thread to  maintain the tk widget `progressbar` itself, then open multi-processes for running the job. Then by receiving communication with the child processes e.g. using shared-memory object or after receiving a result from a process, the thread keeps updating the progressbar, which keeps the progressbar moving.

Another work-around is opening a thread that just keep running *indetermined progressbar* which the bar just keep moving so that user knows the progess is running, this is more easy so I did this.


### Multiprocessing on Windows

When you use `multiprocessing` module on Unix-like/MacOS the process is forked, but on Windows when using `multiporcessing` , the process is not forked is (need to confirm). This creates the same problem when creating a child process, the main Tk will still be duplicated , furthermore creates a lot questionable windows.

The document points out that it is a problem on Windows where we need to use `freeze_support()`.

```py
from multiprocessing import freeze_support
#just before everything
if __name__ == '__main__':
	# before doing anything
	freeze_support()
	...
```

This keeps the `multiprocessing` from creating a lot of duplicated windows.


## misc

跟Tkinter無關但是記一下，是讀取圖片的問題，在使用pillow讀圖片時有一個function是fromarray()，而PIL的範例是給
```py
import PIL.Image
import numpy as np
im = PIL.Image.read('1.jpg')
im = np.asarray(im)
im_pil = PIL.Image.fromarray(im)
```
我當初是把圖片轉成矩陣並且存成list放在另一個python檔案，然後用這個方式讀進來，值得一提的是我存的是黑白圖片，是3 dimension的array，檔案通常會是200\*200\*1這種矩陣，數值是0~255，所以是8-bit的圖片。那我就按照了類似上面的做法
```py
im_pil = PIL.Image.fromarray(np.asarray(array1)) 
# array1 is the array containing the image
```
然後我就嘗試加上mode，但我遇到很奇怪的狀況，就是模式我得選擇16bit的黑白圖片才會正確，但是明明圖片是8bit。然後我就放棄思考繼續用下去，結果之後在另一台電腦執行的時候，讀進來的圖片又壞掉了！
```py
# 8-bit mode is L, which should be the mode I use
im_pil = PIL.Image.fromarray(np.asarray(array1),mode='L')
# 16-bit mode is I
im_pil = PIL.Image.fromarray(np.asarray(array1),mode='I')
```
在處理另一個讀取RBG圖片的時候，因為沒有其他的模式可以選擇(CMYK...)，而且不管怎麼換都一樣，結果終於找到原因，是numpy處理後要記得給它正確的type，不然PIL不認得...。
```py
# 8-bit image
im_pil = PIL.Image.fromarray(np.asarray(array1).astype(np.uint8),mode='I')
# 8-bit RGB image
im_pil = PIL.Image.fromarray(np.asarray(array2).astype(np.uint8),mode='RGB')
# if the type is correct don't even need to set the mode, it will determine it is rgb etc
im_pil = PIL.Image.fromarray(np.asarray(array3).astype(np.uint8))
```