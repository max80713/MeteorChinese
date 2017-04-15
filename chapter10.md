# 透過發佈\(Public\)與訂閱\(Subscribe\)來篩選資料 {#filteringdatawithpublishandsubscribe}

我們已經把敏感的程式碼移動到方法\(Method\)裡，現在我需要學習其他Meteor安全性的措施。到目前為止，我們都把整個資料庫當作是在客戶端，也就是說只要我們呼叫`Tasks.find()`we will get every task in the collection. That's not good if users of our application want to store privacy-sensitive data. We need a way of controlling which data Meteor sends to the client-side database.

Just like with`insecure`in the last step, all new Meteor apps start with the`autopublish`package. Let's remove it and see what happens:

```
meteor remove autopublish
```

When the app refreshes, the task list will be empty. Without the`autopublish`package, we will have to specify explicitly what the server sends to the client. The functions in Meteor that do this are`Meteor.publish`and`Meteor.subscribe`.

First lets add a publication for all tasks:

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

And then let's subscribe to that publication when the`body`template is created:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/cf3557458e16d0477f61cd2cdc5d07adbe225de6)

```
Template.body.onCreated(function bodyOnCreated() {
  this.state = new ReactiveDict();
  Meteor.subscribe('tasks');
});

Template.body.helpers({
```

Once you have added this code, all of the tasks will reappear.

Calling`Meteor.publish`on the server registers a\_publication\_named`"tasks"`. When`Meteor.subscribe`is called on the client with the publication name, the client\_subscribes\_to all the data from that publication, which in this case is all of the tasks in the database. To truly see the power of the publish/subscribe model, let's implement a feature that allows users to mark tasks as "private" so that no other users can see them.

You can read more about publications in the[Data Loading article](http://guide.meteor.com/data-loading.html)of the Meteor Guide.

### Implementing private tasks {#implementingprivatetasks}

First, let's add another property to tasks called "private" and a button for users to mark a task as private. This button should only show up for the owner of a task. It will display the current state of the item.

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

Let's make sure our task has a special class if it is marked private:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/0518f1f4682540652dfabb104fe9f8274ecab735)

```
<template name="task">

<li class="{{#if checked}}checked{{/if}} {{#if private}}private{{/if}}">

<button class="delete">&times;</button>

<input type="checkbox" checked="{{checked}}" class="toggle-checked" />
```

We need to modify our JavaScript code in three places:

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

### Selectively publishing tasks based on privacy status {#selectivelypublishingtasksbasedonprivacystatus}

Now that we have a way of setting which tasks are private, we should modify our publication function to only send the tasks that a user is authorized to see:

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

To test that this functionality works, you can use your browser's private browsing mode to log in as a different user. Put the two windows side by side and mark a task private to confirm that the other user can't see it. Now make it public again and it will reappear!

### Extra method security {#extramethodsecurity}

In order to finish up our private task feature, we need to add checks to our`deleteTask`and`setChecked`methods to make sure only the task owner can delete or check off a private task:

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

> Notice that with this code anyone can delete any public task. With some small modifications to the code, you should be able to make it so that only the owner can delete their tasks.

We're done with our private task feature! Now our app is secure from attackers trying to view or modify someone's private tasks.

