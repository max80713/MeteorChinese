# 更新與刪除待辦事項 {#checkingoffanddeletingtasks}

到目前為止，我們只透過插入文件\(insert documents\)的方式來與資料庫互動。接下來我們將會學習如何更新\(update\)或是刪除\(remove\)文件。

現在，讓我們來修改`task`模板\(template\)吧!首先，我們把它移到獨立的檔案，並且加入一個勾選框\(checkbox\)和一個刪除鈕:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/0f5a29b2ec349b1beefaea65da8795669eb3fbd6)

```
<template name="task">
  <li class="{{#if checked}}checked{{/if}}">
    <button class="delete">&times;</button>

    <input type="checkbox" checked="{{checked}}" class="toggle-checked" />

    <span class="text">{{text}}</span>
  </li>
</template>
```

同時把舊的`task`模板從`imports/ui/body.html`檔案中刪除。

再來加入一些事件處理器\(event handlers\)，來讓勾選框和刪除鈕產生作用:

[imports/ui/task.js»](https://github.com/meteor/simple-todos/commit/9662f35f00dfc5bc0f5b6b363f17324d785b6684)

```
import { Template } from 'meteor/templating';
import { Tasks } from '../api/tasks.js';
import './task.html';

Template.task.events({
  'click .toggle-checked'() {
    // Set the checked property to the opposite of its current value
    Tasks.update(this._id, {
      $set: { checked: !this.checked },
    });
  },
  'click .delete'() {
    Tasks.remove(this._id);
  },
});
```

由於`body`模板使用到`task`模板，所以我們必須載入它:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/95760055fbe30f22a0f05154fbfb70628fa4d80b)

```
import { Tasks } from '../api/tasks.js';

import './task.js';
import './body.html';

Template.body.helpers({
```

### 透過事件處理器\(event handler\)取得資料 {#gettingdataineventhandlers}

在事件處理器中，`this`代表個別的待辦事項物件。在集合\(collection\)中，每一個插入的文件\(document\)都有一個唯一的`_id`欄位，用來代表對應的文件。我們可以透過`this._id`得到該待辦事項的`_id`並且使用`update`或是`remove`來修改對應的待辦事項。

### 更新\(Update\) {#update}

`update`函數接受兩個參數。第一個參數為一個選擇器\(selector\) ，用來篩選符合條件的文件，第二個參數用來指定如何修改符合條件的集合。

以上的程式碼中，選擇器篩選出符合`_id`的待辦事項，然後使用`$set`來切換`checked`欄位，該欄位用來表示此待辦事項是否完成。

### 刪除\(Remove\) {#remove}

`remove`函數接受一個選擇器\(selector\)的參數，用來篩選符合條件的文件，並且從集合中刪除。

### 使用物件屬性或是輔助器來新增或刪除class {#usingobjectpropertiesorhelperstoaddremoveclasses}

加入以上的程式碼之後，試著勾選某個待辦事項，你會發現它被一條刪除線劃掉。這個效果透過以下的程式碼達成:

```
<li class="{{#if checked}}checked{{/if}}">
```

如果`checked`欄位是`true`， `checked`這個class就會被加到待辦事項清單的元素，利用這個class，我們就可以透過CSS來修改完成的待辦事項的樣式。

