# 將待辦事項存入集合\(collection\) {#storingtasksinacollection}

Meteor使用集合\(collections\)來儲存資料。集合最重要的特性就是瀏覽器端和伺服器端都可以對它做存取，我們不需要在伺服器端寫一大堆程式碼就可以將資料呈現在瀏覽器上。除此之外，集合也會自動更新，所以包含集合的模板就會自動呈現最新的資料。

你可以參考[Collections article](http://guide.meteor.com/collections.html)這篇文章來瞭解集合\(collection\)的用法。

要建立一個新的集合非常簡單，只要在JS檔中呼叫`MyCollection = new Mongo.Collection("my-collection");`，在伺服器端就會建立一個叫做`my-collection`的Mongo集合，而在瀏覽器端，則會建立一份連接到伺服器集合的暫存集合。我們在第12章會更深入探討伺服器端和瀏覽器端的區隔 ，目前我們先假裝整個資料庫只存在在瀏覽器端。

我們定義了一個新的模組`task.js`來建立一個Mongo集合並且匯出\(export\)它:

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/9ee57a11d41eac7791920a24497bef8716a6301f)

```
import { Mongo } from 'meteor/mongo';

export const Tasks = new Mongo.Collection('tasks');
```

這邊特別注意，我們另外新建了一個`imports/api`目錄，並且將這個新增的模組放在底下。這個目錄底下專門放置所有的API，包含我們會用到的集合\(collections\)還有之後會加入的發佈\(publications\)以及方法\(methods\)。如果你想了解`import`的作用或是如何管理程式碼檔案的資料夾結構，可以參考Meteor指南裡的[Application Structure article](http://guide.meteor.com/structure.html)。

再來我們在伺服器端載入這個模組，這個動作將會自動為我們新增一個Mongo的集合，並且建立連結來讓瀏覽器取得資料:

[server/main.js»](https://github.com/meteor/simple-todos/commit/96f0e2a82fa51aeb72170c2fb30814245806fd1c)

```
import '../imports/api/tasks.js';
```

然後修改瀏覽器端的JS檔來從集合中取得待辦事項的資料，取代之前從固定的陣列中取得資料的方式:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/f345b40d5bf55026d8bb72a771971d06f6cec25e)

```
import { Template } from 'meteor/templating';
import { Tasks } from '../api/tasks.js';
import './body.html';

Template.body.helpers({
  tasks() {
    return Tasks.find({});
  },
});
```

當你修改完之後，你會發現本來在清單中的待辦事項都不見了!那是因為我們的資料庫目前還是空的，所以我們必須手動輸入一些待辦事項。

### 從「伺服器端資料庫主控台」新增待辦事項 {#insertingtasksfromtheserversidedatabaseconsole}

集合裡的每一筆資料叫做文件\(document\)。我們現在使用伺服器資料庫主控台來新增一些文件到集合\(collection\)裡。打開終端機，進到你的App目錄底下然後輸入:

```
meteor mongo
```

這樣將會打開主控台，並且連到你的App在本地端的資料庫，然後再輸入:

```
db.tasks.insert({ text: "Hello world!", createdAt: new Date() });
```

這個時候頁面上出現一條新的待辦事項!我們不需要寫任何的程式碼來將瀏覽器端連接到伺服器端的資料庫，Meteor會自動幫你更新。

你現在可以透過更改`text`的值來新增幾條不同內容的待辦事項。在下一章，我們將會介紹如何在頁面上加入「新增」的功能，來取代使用資料庫主控台新增的方式。

