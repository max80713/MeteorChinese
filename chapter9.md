# 使用方法\(Methods\)來增加安全性 {#securitywithmethods}

在進行本章節前，任何使用者都可以隨意修改資料庫內的資料，這對於小型的App或Demo App來說是沒有關係的，但實際上應用程式需要控制本身資料庫的存取權限。在Meteor中，解決這個問題的最好辦法就是宣告方法\(declaring _methods_\)，呼叫這些方法時會先檢查目前使用者是否有權限存取資料庫然後再進行存取資料庫的動作，如此可以避免客戶端直接呼叫`insert`，`update`，和`remove`等方法。

### 移除`insecure`套件 {#removinginsecure}

每一個新建的Meteor專案都預設有`insecure`套件，這個套件使得我們在製作App的原型時，可以從客戶端去任修改資料庫， 但現在要製作實際應用的App時，我們必須移除`insecure`套件。進到你的App資料:

```
meteor remove insecure
```

如果在移除這個套件之後你仍繼續使用App，你會發現輸入欄位和按鈕都沒有作用了，這是因為客戶端所有存取資料庫的權限都被關閉了。現在我們需要加入一些程式碼以開啟存取資料庫的權限。

### 定義方法\(Defining methods\) {#definingmethods}

首先，我們定義一些方法。這些方法需要被定義在客戶端和伺服器端都將會被執行的程式碼中，我們會在本章節的最後再作討論。

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/aa0357a3c29f7fdedfbb7ff2b109e990831bb399)

```
import { Meteor } from 'meteor/meteor';
import { Mongo } from 'meteor/mongo';
import { check } from 'meteor/check';

export const Tasks = new Mongo.Collection('tasks');

Meteor.methods({
  'tasks.insert'(text) {
    check(text, String); // Make sure the user is logged in before inserting a task
    if (! Meteor.userId()) {
      throw new Meteor.Error('not-authorized');
    }
    Tasks.insert({
      text,
      createdAt: new Date(),
      owner: Meteor.userId(),
      username: Meteor.user().username,
    });
  },
  'tasks.remove'(taskId) {
    check(taskId, String);
    Tasks.remove(taskId);
  },
  'tasks.setChecked'(taskId, setChecked) {
    check(taskId, String);
    check(setChecked, Boolean);
    Tasks.update(taskId, { $set: { checked: setChecked } });
  },
});
```

定義好方法之後，我們要來使用它:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/f6f479327894e4d8f1cd559cdb587c93a86acb33)

```
const text = target.text.value;

// Insert a task into the collection
Meteor.call('tasks.insert', text);

// Clear form
target.text.value = '';
```

並且移除載入\(`import`\)`Tasks`集合的程式碼:

[imports/ui/task.js»](https://github.com/meteor/simple-todos/commit/574d23454196ee22d0ea3bc6cd152b41e37e42cd)

```
import { Meteor } from 'meteor/meteor';
import { Template } from 'meteor/templating';
import './task.html';

Template.task.events({
  'click .toggle-checked'() {
    // Set the checked property to the opposite of its current value
    Meteor.call('tasks.setChecked', this._id, !this.checked);
  },
  'click .delete'() {
    Meteor.call('tasks.remove', this._id);
  },
});
```

現在，輸入欄位和按鈕又重新有作用了!經過以上的步驟我們達成了以下幾個目的:

1. 當我們要新增待辦事項到資料庫中，我們可以驗證使用者是否登入，`createdAt`、`owner`、`username`等欄位是否正確。
2. 之後我們可以增加待辦事項擁有者的檢查機制，這樣使用者就只能針對其所新增的待辦事項呼叫`setChecked`或`remove。`
3. 將客戶端的程式碼和資料庫存取程式碼分開，避免在事件處理器\(event handler\)中出現複雜且龐大的程式碼，除此之外，不管在客戶端或是伺服器端我們都可以重複使用這些我們定義好的方法。

### 使用者介面最佳化\(Optimistic UI\) {#optimisticui}

所以，為什麼我們要把方法定義在客戶端和伺服器端都可以運行的檔案中？我們這麼做是為了使用者介面最佳化。

當我們在客戶端透過`Meteor.call`來呼叫方法，有兩件事情會同時進行:

1. 客戶端會發送請求\(request\)到伺服器端，要求伺服器端在安全的環境下執行該方法，就像AJAX請求一樣。
2. 客戶端會模擬伺服器端執行該方法，並且嘗試預測伺服器執行的結果。

這樣一來，在客戶端收到從伺服器端回傳的結果前，新增的待辦事項便會顯示在畫面上。

如果伺服器端回傳的結果和客戶端模擬出來的結果是一致的，則不會有任何改變。而如果伺服器端回傳的結果和客戶端模擬出來的結果是不一致的，使用者介面便會自動更新為正確的結果。

你可以參考[Methods article](http://guide.meteor.com/methods.html)和[blog post about optimistic UI](http://info.meteor.com/blog/optimistic-ui-with-meteor-latency-compensation)來更深入瞭解方法\(Method\)和使用者介面最佳化\(Optimistic UI\)的運作原理。

