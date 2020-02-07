---
layout: post
title:  開發紀錄1
---

之前弄github pages還有遠大的志向想要用react自己刻一個app出來。
但是摸魚了一個學期以後，進度還是0就知道不可能了...

所以就用github pages官方推薦的`jekyll`進行了，第一篇是`jekyll`可以看出來XD
目前還在摸要怎麼改造，總之為了有點成就感所以記錄下。

我是在macOS上面開發的，然後官方是要求`ruby2.4`以上，我用`brew`更新以後發現原生的系統就有安裝ruby了，但是2.3版本的。因為在解決的時候意外問到Catalina的預設就是`ruby2.6`，所以就索性乾脆更新成10.15Catalina。

接下來就依照github pages的[官網](https://help.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll)安裝，結果遇到大困難。

`gem install --user-install bundler jekyll`

因為不熟悉`ruby`的特性，所以我完全搞不懂下載`bundle`	的用意是啥（即使文件有解釋）

`bundle exec jekyll new .`

這部分的用意是請bundle執行`jekyll new`這個指令，但是我第一次執行的時候一直出現

`No Gemfile`

這個錯誤，最後我想乾脆就不依照文件的意思直接執行

`jekyll new .`

然後出現新的錯誤（崩潰），主要是說這個資料夾底下已經有資料了，不能執行。
然後我刪掉資料夾下的所有東西後`rm -r *`重複了上述有出現了錯誤。
最後我在清空後直接下`jekyll new .`才成功。

原因是文件指"if you have bundler installed"應該是指有下過`bundle install`這個指令，並不是`gem install bundler`安裝了bundler。
而`jekyll new`的第一步就是幫你執行了`bundle install`安裝了需要的gem(ruby下的模組)並且幫你寫好了Gemfile，下次就可以執行`bundle exec jekyll new .`
上述的狀況，我在執行`bundle install`前忘記清空，導致`bundle exec jekyll new .`的時候出現資料夾必須是空的錯誤訊息。

最後依照`bundle`的指示重新下了`bundle update`將太新的套件下降版本，他會幫你處理好。

---

總結
1. 安裝ruby(用brew或是官方都可以)
2. `gem install --user-install bundler jekyll`
3. 進入github page的目錄
4. 清空目錄（避免`jekyll new .`出現錯誤）
5. - 第一次指令下 `jekyll new .` （比較萬用）
	- 或是`bundle install`之後再下`bundle exec jekyll new .`（bundle會幫你掌握套件的衝突比較安全）
6. `bundle update`
7. `bundle exec jekyll serve` 這樣就架起來了。