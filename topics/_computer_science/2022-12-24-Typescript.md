---
title: Typescript
tags: frontend, typescript
---    
會寫這篇純粹是為了記錄之前訓練的內容。

## Whats the point
大致上是
- Kotlin(我需要學)
- React
- funcitonal programming
- Type safe: Typescript + Kotlin(strong type)
- k8s: k8s(基本知識) + ArgoCD(用法)
- Error Handling

有些(教的東西)太基礎了，真的是給全新的coding人
- git
- Tools: VScode、IntelliJ、Maven、npm、WSL
- docker
- MongoDB

另外一些比較像是文化的部分、或是好的軟體的設計部分
- Agile: Scrum
- DevOps
- TDD [Trest Driven Devolopment] (Me: Is this a good practice?)
- System Design
- Root Cause Analysis (RCA)
- Micro-service (這個架構真的是建議在某些地才適用)

(附註:編輯這篇文章時已經新訓結束後一年了)

## Typescript

> 當初寫這篇的時候是為了紀錄一些從 js 轉到 ts 時遇到的問題，但是因為現在都已經有點久遠了，已經寫 ts 好一段時間，所以現在記錄下的問題可能相對是比較熟悉才會遇到的問題，而不是新訓新手會遇到的問題。
> 
> 不過還是總結一下差異好了

### 特性

Typescript 的特色就是強型別，以我目前的經驗來說，覺得整體還是不錯的，可以幫助我們了解正在傳來傳去的資料長怎麼樣，不需要求助其他開發才可以了解資料長怎麼樣。但因為我開始寫 ts 之後沒有可以嘗試新的 js 的機會，之前小小碰一下的時候發現 js 好像也有一點點支援 type ，所以搞不好 ts 的優勢沒有相對那麼大了。

**優點**
- 強型別
- 龐大的專案、架構下，清楚的型別確實很有幫助，尤其跟後端對接的 json ，可以有很詳細的 interface
- 會在服務跑起來前 compile 來檢查是否有 Type error，可以檢查到大部分的錯誤。

**缺點**
- Runtime 時該發生的 error 仍然會發生。
- type check 有時候很惱人，產生冗長的 interface 前綴
- 對於某些 `object` 可以用 `array` 的function之類的，因為 type 的關係會 compile error
- 使用 commonjs 的套件時，需要花時間幫忙處理 type 的問題。不然只能全部 `any` 或是忽略。

### 問題
以現在工作上會遇到的問題，最多的就是 `type` 跟 `interface` 相關的問題了。

(待補上，先發出文章以逼迫自己更新)