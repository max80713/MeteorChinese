# 將使用者介面狀態儲存於響應式字典\(Reactive Dictionary\) {#storingtemporaryuistateinareactivedictionary}

In this step, we'll add a client-side data filtering feature to our app, so that users can check a box to only see incomplete tasks. We're going to learn how to use a[`ReactiveDict`](https://atmospherejs.com/meteor/reactive-dict)to store temporary reactive state on the client. A`ReactiveDict`is like a normal JS object with keys and values, but with built-in reactivity.

First, we need to add a checkbox to our HTML:

**7.1**

Add hide-completed checkbox to HTML

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

Now we need to add the`reactive-dict`package:

```
meteor add reactive-dict
```

Then we need to set up a new`ReactiveDict`and attach it to the body template instance \(as this is where we'll store the checkbox's state\) when it is first created:

**7.3**

Add state dictionary to the body

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

Then, we need an event handler to update the`ReactiveDict`variable when the checkbox is checked or unchecked. An event handler takes two arguments, the second of which is the same template instance which was`this`in the`onCreated`callback:

**7.4**

Add event handler for checkbox

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



