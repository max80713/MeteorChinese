# 將使用者介面狀態暫存於響應式字典\(Reactive Dictionary\) {#storingtemporaryuistateinareactivedictionary}

在這個章節，我們將在使用者介面加入篩選功能，讓使用者可以篩選出未完成的待辦事項。我們將學習如何使用響應式字典\([`ReactiveDict`](https://atmospherejs.com/meteor/reactive-dict)\)來暫存客戶端的響應式狀態。`ReactiveDict`就像一般的JavaScript物件，有鍵\(Key\)與對應的值\(Value\)，除此之外，還內建有響應式的特性。

首先，我們在HTML檔裡加入一個勾選框:

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/4bfaa6f070101cb0caf35ab30343e1126b0e6701)

```
<header>
  <h1>Todo List</h1>
  <label class="hide-completed">
    <input type="checkbox" />Hide Completed Tasks
  </label>

  <form class="new-task">
    <input type="text" name="text" placeholder="Type to add new tasks" />
  </form>
```

再來我們需要加入`reactive-dict`套件:

```
meteor add reactive-dict
```

然後設置一個新的`ReactiveDict`並且在body模板物件第一次被建立的時候附加上去，我們將會把勾選框的狀態存在這個`ReactiveDict`中:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/349bd90805ba098d08c9445c00fb6776f2cb8b08)

```
import { Template } from 'meteor/templating';
import { ReactiveDict } from 'meteor/reactive-dict';
import { Tasks } from '../api/tasks.js';
import './task.js';
import './body.html';

Template.body.onCreated(functionbodyOnCreated() {
  this.state = new ReactiveDict();
});

Template.body.helpers({
  tasks() {
  // Show newest tasks at the top
```

之後我們需要一個事件處理器\(event handler\)，當勾選框被勾選或取消勾選的時候來更新`ReactiveDict`變數。一個事件處理器接收兩個參數，其中第二個參數和`onCreated`函數中的`this`所代表的是同樣的模板物件:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/caa11a11d808123299380ee26229c9f358ba1775)

```
    // Clear form
    target.text.value = '';
  },

  'change .hide-completed input'(event, instance) {
    instance.state.set('hideCompleted', event.target.checked);
  },
});
```

接著我們需要修改一下`Template.body.helpers`，以下的程式碼中新增了一個if區塊，當勾選框被勾選的時候便會篩選待辦事項:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/10e30f2ff2b42a53bd675433f65d21ac2beb679e)

```
Template.body.helpers({
  tasks() {
    const instance = Template.instance();
    if(instance.state.get('hideCompleted')) {
      // If hide completed is checked, filter tasks
      return Tasks.find({ checked: { $ne: true } }, { sort: { createdAt: -1 } });
    }
    // Otherwise, return all of the tasks
    return Tasks.find({}, { sort: { createdAt: -1 } });
  },
});
```

現在只要我們點擊勾選框，待辦事項清單就只會顯示未完的待辦事項囉!

### 響應式字典 {#reactivedictsarereactivedatastoresfortheclient}

到目前為止，我們將資料儲存於集合\(collections\)中，當我們修改集合中的資料時，網頁畫面就會自動更新。這是因為Mongo的集合為響應式的資料庫，也就是說只要資料有變動Meteor就會知道。`ReactiveDict`也是同樣的原理，但和集合不同的地方在於`ReactiveDict`並非像集合一樣與伺服器端同步，這讓`ReactiveDict`本身擁有可以暫時儲存使用者介面狀態的優點，就好比我們剛剛使用它來儲存勾選框的狀態，同時它也擁有像集合一樣的優點，我們不需要再寫額外的程式碼來當`ReactiveDict`的變數改變時主動去更新模板，我們只需要在輔助器\(helper\)中呼叫`instance.state.get(...)`即可。

如果想更深入了解Meteor的設計模式，可以參考[Blaze article](http://guide.meteor.com/blaze.html)。

### 顯示未完成待辦事項的數量 {#onemorefeatureshowingacountofincompletetasks}

我們剛寫好篩選未完成待辦事項的功能，再來我們可以用類似的方式來取得未完成待辦事項的數量，只要多加一個輔助器\(helper\)並且修改其中一行即可。

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/79b34c54716abd5aaa1a5d9f5068a8bd7c24e35b)

```
    // Otherwise, return all of the tasks
    return Tasks.find({}, { sort: { createdAt: -1 } });
  },

  incompleteCount() {
    return Tasks.find({ checked: { $ne: true } }).count();
  },
});

Template.body.events({
```

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/b26b4d486c9136a3db7beb5d63759b7fa1cdf0b3)

```
<body>
  <div class="container">
    <header>
      <h1>Todo List ({{incompleteCount}})</h1>
      <label class="hide-completed">
      <input type="checkbox" />
```



