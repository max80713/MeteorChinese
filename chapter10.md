# 透過發佈\(Publish\)與訂閱\(Subscribe\)來篩選資料 {#filteringdatawithpublishandsubscribe}

我們已經把敏感的程式碼移動到方法\(Method\)裡，現在我需要學習其他Meteor提升安全性的措施。到目前為止，我們都把整個資料庫當作是在客戶端，也就是說只要我們呼叫`Tasks.find()`，我們會從集合\(collection\)中得到所有的待辦事項。如果使用者想要儲存一些私密的資料將會非常沒有隱私，我們需要控制哪些資料會被送到客戶端的資料庫。

就像上一個章節提到的`insecure`套件，所有Meteor的App也都預設有`autopublish`套件.。我們試著移除它並且看看會發生什麼事:

```
meteor remove autopublish
```

當我們重新整理網頁，待辦事項都不見了!沒有了`autopublish`這個套件，我們就必須透過`Meteor.publish`和`Meteor.subscribe`來告訴伺服器端要送那些資料到客戶端。

首先，我們試著發佈\(publish\)所有的待辦事項:

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/09284b4286add29217f39a59c7b7c63b93d6a74f)

```
export const Tasks = new Mongo.Collection('tasks');

if (Meteor.isServer) {
  // This code only runs on the server
  Meteor.publish('tasks', functiontasksPublication() {
    return Tasks.find();
  });
}

Meteor.methods({
  'tasks.insert'(text) {
    check(text, String);
```

當`body`模板建立完成之後，訂閱\(subscribe\)所有的待辦事項:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/cf3557458e16d0477f61cd2cdc5d07adbe225de6)

```
Template.body.onCreated(function bodyOnCreated() {
  this.state = new ReactiveDict();
  Meteor.subscribe('tasks');
});

Template.body.helpers({
```

只要增加以上的程式碼，所以的待辦事項會重新出現。

在伺服器端呼叫`Meteor.publish`會建立一個發佈叫`"tasks"`，然後在客戶端呼叫`Meteor.subscribe`並且傳入發佈的名字作為參數，客戶端將訂閱所有發佈的資料，在這個例子中這些資料就是資料庫中所有的待辦事項。接下來我們嘗試讓使用者可以將待辦事項設為"非公開"\(private\)，讓其他使用者看不到該待辦事項。

如果你想深入了解發佈\(publications\)，可以參考[Data Loading article](http://guide.meteor.com/data-loading.html)。

### 將待辦事項設為非公開 {#implementingprivatetasks}

首先，我們在每個待辦事項顯示"private"或是"public"來告訴使用者該待辦事項是否為公開的，並且在頁面上新增一個按鈕讓使用者來將待辦事項設為非公開。這個按鈕只顯示給該待辦事項的擁有者看到:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/559e0285e8d2ff713e00997b6f95463433fffa40)

```
    <input type="checkbox" checked="{{checked}}" class="toggle-checked" />
      {{#if isOwner}}
        <button class="toggle-private">
          {{#if private}}
            Private
          {{else}}
            Public
          {{/if}}
        </button>
      {{/if}}

    <span class="text"><strong>{{username}}</strong> - {{text}}</span>

  </li>
</template>
```

如果該待辦事項被標記為非公開，就將它加入一個private的類別:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/0518f1f4682540652dfabb104fe9f8274ecab735)

```
<template name="task">

<li class="{{#if checked}}checked{{/if}} {{#if private}}private{{/if}}">

<button class="delete">&times;</button>

<input type="checkbox" checked="{{checked}}" class="toggle-checked" />
```

接下來在三個不同的地方修改我們JavaScript的程式碼:

[imports/ui/task.js»](https://github.com/meteor/simple-todos/commit/5f5dfb0eda840c8e796e07696e8412385453ead6)

```
import './task.html';

Template.task.helpers({
  isOwner() {
    return this.owner === Meteor.userId();
  },
});

Template.task.events({
  'click .toggle-checked'() {
  // Set the checked property to the opposite of its current value
```

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/f3a0faebe7b53432e719148d109202a3179e207d)

```
    Tasks.update(taskId, { $set: { checked: setChecked } });
  },
  'tasks.setPrivate'(taskId, setToPrivate) {
    check(taskId, String);
    check(setToPrivate, Boolean);
    const task = Tasks.findOne(taskId);
    // Make sure only the task owner can make a task private
    if (task.owner !== Meteor.userId()) {
      throw new Meteor.Error('not-authorized');
    }
    Tasks.update(taskId, { $set: { private: setToPrivate } });
  },
});
```

[imports/ui/task.js»](https://github.com/meteor/simple-todos/commit/e666e1411463ac2e076bd6b966c6ec9c85b504bd)

```
  'click .delete'() {
    Meteor.call('tasks.remove', this._id);
  },
  'click .toggle-private'() {
    Meteor.call('tasks.setPrivate', this._id, !this.private);
  },
});
```

### 根據權限來發佈待辦事項 {#selectivelypublishingtasksbasedonprivacystatus}

現在使用者已經可以將待辦事項標記為非公開，再來我們要修改我們的發佈函數\(publication function\)來告訴伺服器端只將使用者有權限可以看到的待辦事項送到客戶端:

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/1501ba07e7032887345eddef0fe542bfc8a21283)

```
if (Meteor.isServer) {
  // This code only runs on the server
  // Only publish tasks that are public or belong to the current user
  Meteor.publish('tasks', function tasksPublication() {
    return Tasks.find({
      $or: [
        { private: { $ne: true} },
        { owner: this.userId },
      ],
    });
  });
}
```

你可以使用瀏覽器的無痕模式來開啟兩個視窗，並且用不同的使用者帳號登入來測試這個功能。將這兩個視窗擺在一起，然後在其中一個視窗將某個待辦事項設為非公開，這個待辦事項在另外一個視窗就會被隱藏起來，如果重新設為公開就會重新出現囉!

### 加入更完整的安全措施 {#extramethodsecurity}

我們在`deleteTask`和`setChecked`這兩個方法\(method\)中加入一些檢查機制以確保每個待辦事項只有擁有者有權限刪除或是將該待辦事項設為已完成:

[imports/api/tasks.js»](https://github.com/meteor/simple-todos/commit/b47254b75a77b7e17374a0cfefaa2b491db047bf)

```
  'tasks.remove'(taskId) {
    check(taskId, String);
    const task = Tasks.findOne(taskId);
    if (task.private && task.owner !== Meteor.userId()) {
      // If the task is private, make sure only the owner can delete it
      throw new Meteor.Error('not-authorized');
    }
    Tasks.remove(taskId);
  },
  'tasks.setChecked'(taskId, setChecked) {
    check(taskId, String);
    check(setChecked, Boolean);
    const task = Tasks.findOne(taskId);
    if (task.private && task.owner !== Meteor.userId()) {
      // If the task is private, make sure only the owner can check it off
      throw new Meteor.Error('not-authorized');
    }
    Tasks.update(taskId, { $set: { checked: setChecked } });
  },
  'tasks.setPrivate'(taskId, setToPrivate) {
```

> 注意:即使加入以上的程式碼，每個人還是有權限可以刪除公開的待辦事項。你可以嘗試自行修改程式碼以確保每個公開或是非公開的待辦事項，都只有擁有者可以刪除它。



