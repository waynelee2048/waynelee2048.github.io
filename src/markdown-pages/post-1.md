---
slug: "/2020/07/06/building-the-page-at-runtime"
date: "2020-07-06"
title: "JS 忍者 CH2. 在執行時期產生網頁"
tags: ["JavaScript", "reading notes", ""]
---
讀書的隨手筆記與延伸問題  

書籍：[忍者：JavaScript 開發技巧探秘（第二版)](https://www.books.com.tw/products/0010773867)
### 2.1 生命週期概述
  1. 建立頁面階段
  2. 事件處理階段

### 2.2 頁面建立階段
  1. 解析 HTML 並建立 DOM
  2. 執行 JavaScript 程式

瀏覽器在頁面建立階段時，會根據需求在步驟 1 與 步驟 2 之間切換許多次

---
問：在第二階段時，已建立完的 DOM 會顯示在瀏覽器上嗎?   
答：會
```html
<h1>First</h1>

<script>
    var i = 0; // 此時 First 的文字會顯示於頁面
</script>

<h1>Two</h1>

<script>
    console.log(i); // 此時 Two 的文字會顯示於頁面
</script>

<h1>Three</h1>
```
---
#### 2.2.1 頁面建立階段
遇到 script element 時，會暫停解析 HTML，開始執行 JavaScript 程式。

瀏覽器可修復在 HTML 上發現的問題，以建立一個有效的 DOM。

---
工作到現在，有時候在討論的過程中，有些人會講 element，有些人會講 tag，決定要好好來個名詞解釋

[An HTML element is defied by a start tag, some content, and an end tag.](https://www.w3schools.com/html/html_elements.asp)
```html
<script> <!-- start tag -->
	function() {}; // content
</script> <!-- end tag -->

<h1> <!-- start tag -->
	Content <!-- content -->
</h1> <!-- end tag -->
```
期望自己以後要講的明確 XD

---
#### 2.2.2 執行 JavaScript 程式
JavaScript 核心 (以 ECMAScript 標準為基礎)

* BOM (Browser Object Model，瀏覽器物件模型)
* DOM (Document Object Model，文件物件模型)

BOM 就是 window  
DOM 就是 window.document

在頁面建立階段，JavaScript 無法選取與修改尚未建立的 element，這就是為什麼傾向把 script 放在 body 的最後，確保所有的 element 都解析並建立成 DOM

在頁面建立階段，JavaScript 會維持他的全域狀態
```html
<h1>Hi</h1>

<script>
	var iStillLive = 'love';
</script>

<h1>Hello</h1>

<script>
	console.log(iStillLive); // 'love'
</script>
```

---
問：那在不同的 Script element 之間，也會有 hoisting 的情形嗎?  
答：會出現 ReferenceError

```html
<script> // 第一個 script element
	console.log('error', hoist); // Uncaught ReferenceError: hoist is not defined
</script>
<script> // 我是第二個
	console.log('hoist is undefined', hoist); // undefined;
	var hoist = 'YOOOO';
</script>
```

個人覺得這種情境很有趣，可能是因為瀏覽器在執行第一個 script element 的程式時，會暫停 DOM 的建置，所以第二個 script  沒有被建置與執行。  

這也是為什麼我們在使用 plugin 時，必須要注意載入 script 的先後順序。。
```html
<!-- jQuery -->
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<!-- FlexSlider -->
<script defer src="js/jquery.flexslider.js"></script>
```

---
### 2.3 事件處理

#### 2.3.1 事件處理概述
瀏覽器執行環境的核心是基於一次只能執行一段程式碼的觀念，也就是單一執行緒執行模型(single-threaded execution model)

*The browser execution environment is, at its core, based on the idea that only a single piece of code can be executed at once: the so-called *single-threaded* execution model.*

情境：銀行只有一個窗口，一次只能處理一個客人，每個客人都要排隊。

使用者產生的事件或是伺服器產生的事件，都會按照順序放到 event queue，由瀏覽器進行檢測

將事件放入 event queue 的瀏覽器工作機制，不在頁面建立與事件處理的執行緒之中

---
問：瀏覽器有幾個 process 與 thread 在運行?  
答：同時有多個 process 在運行

* Browser process：主控瀏覽器相關，像是書籤功能、發出網路請求等
	* UI thread
	* Network thread
* Renderer process：頁面顯示
	* Main thread
	* Worker thread
	* Compoisitor thread
	* Raster thread 
* Plugin process：控制網站所用的 plugin，例如：flash
* GPU process：跨核心的工作與 3D 圖形繪製相關等

每個 tab 擁有自己獨立運作的（renderer）process。

每個 iframe 是由獨立運作的 renderer process 來負責執行

Browser process 與 Renderer process 之間透過 IPC ( inter process communication ) 進行溝通

---
事件是非同步的

事件類型
* 使用者事件：點擊、鍵盤輸入、移動滑鼠
* 瀏覽器事件：網頁處於已載入或未載入的狀態
* 網路事件：來自伺服器的回應 ( ajax )
* 計時器事件：setTimeout 或 setInterval

事件處理的概念是 Web 應用程式的核心，絕大多數的程式碼都將作為某些事件的結果而被執行

#### 2.3.2 註冊事件處置器
註冊事件的方式
* 將函式指定給特殊屬性
* 使用內建的 addEventListener 方法

```js
window.onload = function() {}; // 將 function 指定給 onload 屬性
document.body.onclick = function() {}; // 將 function 指定給 onload 屬性
// 缺點，某一個特定事件只能指派一個事件處置器
// It’s only possible to register one function handler for a particular event.
// 容易覆蓋掉之前的事件處理函式

document.body.addEventListener('mousemove', function() {}); 
document.body.addEventListener('click', function() {});
```

事件迴圈 ( Event Loop )會一直執行，直到使用者關閉 Web 應用程式

## References
[從內部來看瀏覽器到底在做什麼？](https://cythilya.github.io/2018/11/10/inside-look-at-modern-web-browser/)  
[重學瀏覽器(1)-多程式多執行緒的瀏覽器](https://iter01.com/429161.html)  
[Alfredo's Note](https://www.notion.so/Chapter2-126e9aaf8bf142fb8e2f4a6e38d431c8)  
[Austin's Note ](https://rockonyu.github.io/2020/07/JS%20%E5%BF%8D%E8%80%85%E8%AE%80%E5%BE%8C%E9%9A%A8%E7%AD%86%20CH1%20&%20CH2/)


<br>
<br>
<br>