# 第一支App

本教學的目標是要帶領大家做出一個可以同時多人使用的"待辦事項管理App"。

首先要建立一個範本App，非常簡單，只要打開終端機然後輸入:

```
meteor create simple-todos
```

這樣就會自動建立一個資料夾叫做`simple-todos`，資料夾底下包含所有啟動並且執行一個Meteor App所需要的檔案:

```
client/main.js        
# 瀏覽器進入點

client/main.html      
# 負責定義網頁模板的HTML檔

client/main.css       
# 負責定義網頁樣式的CSS檔

server/main.js        
# 伺服器進入點

package.json          
# 安裝各種NPM套件的設定檔

.meteor               
# Meteor內部檔案

.gitignore            
# Git設定檔
```

第一次執行App:

```
cd simple-todos
meteor npm install
meteor
```

打開瀏覽器然後輸入`http://localhost:3000`就可以看到這個App已經在執行了!

現在你可以試著修改這個App。打開文字編輯器，把`client/main.html`裡面的`<h1>`改成你喜歡的樣式，例如:`<h4>`然後存檔，再回去看正在執行的App，你會發現字體變小了!這是因為當App正在執行的時候，Meteor會偷偷在背後觀察你的程式碼，如果有發現任何修改，Meteor會自動幫你更新，我們稱之「hot code push」。

### ES2015 JavaScript 特性 {#es2015javascriptfeatures}

在這個範本App的檔案中或是在接下來的教學中，你可能會看到一些不熟悉的JavaScript語法，那是因為Meteor本身搭載並且支援ES2015 JavaScript的新語法，包含以下特性:

1. 箭頭函數: `(arg) => {return result;}`
2. 簡式函數: `render() { ... }`
3. 使用 `const` 和 `let` 取代 `var`

所有ES2015 JavaScript的特性都可以在[ecmascript docs](https://docs.meteor.com/#/full/ecmascript)找到。或是可以參考神人更詳細的介紹:

* [Luke Hoban's "ES6 features"](http://git.io/es6features)

* [Kyle Simpson's "You don't know JS: ES6 and beyond"](https://github.com/getify/You-Dont-Know-JS/tree/master/es6%20%26%20beyond)

* [Nikolas C. Zakas "Understanding ECMAScript 6"](https://github.com/nzakas/understandinges6)

現在你已經會簡單的修改了，接下來會帶各位一步步來完成我們的"待辦事項管理App"。

