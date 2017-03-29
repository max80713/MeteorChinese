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

Now, we need to update`Template.body.helpers`. The code below has a new if block to filter the tasks if the checkbox is checked:

**7.5**

Add helpers to body template

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

Now if you check the box, the task list will only show tasks that haven't been completed.

### ReactiveDicts are reactive data stores for the client {#reactivedictsarereactivedatastoresfortheclient}

Until now, we have stored all of our state in collections, and the view updated automatically when we modified the data inside these collections. This is because Mongo.Collection is recognized by Meteor as a_reactive data source_, meaning Meteor knows when the data inside has changed.`ReactiveDict`is the same way, but is not synced with the server like collections are. This makes a`ReactiveDict`a convenient place to store temporary UI state like the checkbox above. Just like with collections, we don't have to write any extra code for the template to update when the`ReactiveDict`variable changes — just calling`instance.state.get(...)`inside the helper is enough.

You can read more about patterns for writing components in the[Blaze article](http://guide.meteor.com/blaze.html)of the Meteor Guide.

### One more feature: Showing a count of incomplete tasks {#onemorefeatureshowingacountofincompletetasks}

Now that we have written a query that filters out completed tasks, we can use the same query to display a count of the tasks that haven't been checked off. To do this we need to add a helper and change one line of the HTML.

**7.6**

Add incompleteCount helper to body

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

**7.7**

Display incompleteCount

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/b26b4d486c9136a3db7beb5d63759b7fa1cdf0b3)

```
<body>
  <div class="container">
    <header>
      <h1>Todo List ({{incompleteCount}})</h1>
      <label class="hide-completed">
      <input type="checkbox" />
```



