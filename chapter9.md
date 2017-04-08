# 使用方法來增加安全性 {#securitywithmethods}

在進行本章節前，任何使用者都可以隨意修改資料庫內的資料，這對於小型的App或Demo App來說是沒有關係的，但實際上應用程式需要控制本身資料庫的存取權限。在Meteor中，解決這個問題的最好辦法就是宣告方法\(declaring _methods_\)，呼叫這些方法時會先檢查目前使用者是否有權限存取資料庫然後再進行存取資料庫的動作，如此可以避免客戶端直接呼叫`insert`，`update`，和`remove`等方法。

### 移除`insecure`套件 {#removinginsecure}

Every newly created Meteor project has the`insecure`package added by default. This is the package that allows us to edit the database from the client. It's useful when prototyping, but now we are taking off the training wheels. To remove this package, go to your app directory and run:

```
meteor remove insecure
```

If you try to use the app after removing this package, you will notice that none of the inputs or buttons work anymore. This is because all client-side database permissions have been revoked. Now we need to rewrite some parts of our app to use methods.

### Defining methods {#definingmethods}

First, we need to define some methods. We need one method for each database operation we want to perform on the client. Methods should be defined in code that is executed on the client and the server - we will discuss this a bit later in the section titled_Optimistic UI_.

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
    }Tasks.insert({
```

```
     text,
```

```
      createdAt: 
new
Date
(),
```

```
      owner: Meteor.userId(),
```

```
      username: Meteor.user().username,
```

```
    });
```

```
  },
```

```
'tasks.remove'
(taskId) {
```

```
    check(taskId, 
String
);
```

```
    Tasks.remove(taskId);
```

```
  },
```

```
'tasks.setChecked'
(taskId, setChecked) {
```

```
    check(taskId, 
String
);
```

```
    check(setChecked, 
Boolean
);
```

```
    Tasks.update(taskId, { $set: { checked: setChecked } });
```

```
  },
```

```
});
```

Now that we have defined our methods, we need to update the places we were operating on the collection to use the methods instead:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/f6f479327894e4d8f1cd559cdb587c93a86acb33)

```
const
 text = target.text.value;
```

```
// Insert a task into the collection
```

```
    Meteor.call(
'tasks.insert'
, text);
```

```
// Clear form
```

```
    target.text.value = 
''
;
```

We do the same on this file, but also remove the`import`of the`Tasks`collection since it's no longer necessary:

[imports/ui/task.js»](https://github.com/meteor/simple-todos/commit/574d23454196ee22d0ea3bc6cd152b41e37e42cd)

```
import { Meteor } from 
'meteor/meteor'
;
```

```
import { Template } from 
'meteor/templating'
;
```

```
import 
'./task.html'
;
```

```
Template.task.events({
```

```
'click .toggle-checked'
() {
```

```
// Set the checked property to the opposite of its current value
```

```
    Meteor.call(
'tasks.setChecked'
, 
this
._id, !
this
.checked);
```

```
  },
```

```
'click .delete'
() {
```

```
    Meteor.call(
'tasks.remove'
, 
this
._id);
```

```
  },
```

```
});
```

Now all of our inputs and buttons will start working again. What did we gain from all of this work?

1. When we insert tasks into the database, we can now securely verify that the user is logged in, that the
   `createdAt`
   field is correct, and that the
   `owner`
   and
   `username`
   fields are correct and the user isn't impersonating anyone.
2. We can add extra validation logic to
   `setChecked`
   and
   `remove`
   in later steps when users can make tasks private.
3. Our client code is now more separated from our database logic. Instead of a lot of stuff happening inside our event handlers, we now have methods that can be called from anywhere.

### Optimistic UI {#optimisticui}

So why do we want to define our methods on the client and on the server? We do this to enable a feature we call_optimistic UI_.

When you call a method on the client using`Meteor.call`, two things happen in parallel:

1. The client sends a request to the server to run the method in a secure environment, just like an AJAX request would work
2. A simulation of the method runs directly on the client to attempt to predict the outcome of the server call using the available information

What this means is that a newly created task actually appears on the screen\_before\_the result comes back from the server.

If the result from the server comes back and is consistent with the simulation on the client, everything remains as is. If the result on the server is different from the result of the simulation on the client, the UI is patched to reflect the actual state of the server.

You can read more about methods and optimistic UI in the[Methods article](http://guide.meteor.com/methods.html)of the Meteor Guide, and our[blog post about optimistic UI](http://info.meteor.com/blog/optimistic-ui-with-meteor-latency-compensation).

