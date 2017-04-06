# 新增使用者帳號 {#addinguseraccounts}

Metoer內建帳號系統和登入系統使用者介面來讓你快速地完成會員系統的功能。

我們必須載入相關套件來開啟這些功能。在你的App資料夾底下，輸入:

```
meteor add accounts-ui accounts-password
```

在HTML檔裏頭，加入以下的程式碼來在頁面上新增一個登入的下拉式選單:

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/bc9fb936699c1ce8a0643e5c563043d560a04811)

```
        Hide Completed Tasks
      </label>

      {{> loginButtons}}

      <form class="new-task">
        <input type="text" name="text" placeholder="Type to add new tasks" />
      </form>
```

Then, in your JavaScript, add the following code to configure the accounts UI to use usernames instead of email addresses:

[imports/startup/accounts-config.js»](https://github.com/meteor/simple-todos/commit/7c48c9aa89e26eac39cc67046f85e54bab5889fe)

```
import { Accounts } from 'meteor/accounts-base';
Accounts.ui.config({
  passwordSignupFields: 'USERNAME_ONLY',
});
```

We'll need to import that configuration from our\_client-side JavaScript entrypoint\_also:

[client/main.js»](https://github.com/meteor/simple-todos/commit/47fde1a42d5d6d1b765b2f16d0f0cc48e0567be1)

```
import '../imports/startup/accounts-config.js';
import '../imports/ui/body.js';
```

Now users can create accounts and log into your app! This is very nice, but logging in and out isn't very useful yet. Let's add two functions:

1. Only display the new task input field to logged in users
2. Show which user created each task

To do this, we will add two new fields to the`tasks`collection:

1. `owner` - the `_id` of the user that created the task.
2. `username` - the `username` of the user that created the task. We will save the username directly in the task object so that we don't have to look up the user every time we display the task.

First, let's add some code to save these fields into the`submit .new-task`event handler:

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/2e4234a228346ca731731166ca12aa38c857d82d)

```
import { Meteor } from 'meteor/meteor';
import { Template } from 'meteor/templating';
import { ReactiveDict } from 'meteor/reactive-dict';

...some lines skipped...
  Tasks.insert({
    text,
    createdAt: new Date(), // current time
    owner: Meteor.userId(),
    username: Meteor.user().username,
  });

// Clear form
```

Then, in our HTML, add an`#if`block helper to only show the form when there is a logged in user:

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/7083c5b56ba521ed7f34a7039bb3510e6f522534)

```
      {{> loginButtons}}
      {{#if currentUser}}
        <form class="new-task">
          <input type="text" name="text" placeholder="Type to add new tasks" />
        </form>
      {{/if}}
    </header>

    <ul>
```

Finally, add a Spacebars statement to display the`username`field on each task right before the text:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/da75b1705c5d5ae3470f47406c261d4303f95a87)

```
    <input type="checkbox" checked="{{checked}}" class="toggle-checked" />

    <span class="text"><strong>{{username}}</strong> - {{text}}</span>
  </li>
</template>
```

Now, users can log in and we can track which user each task belongs to. Let's look at some of the concepts we just discovered in more detail.

### Automatic accounts UI {#automaticaccountsui}

If our app has the`accounts-ui`package, all we have to do to add a login dropdown is include the`loginButtons`template with`{{>loginButtons}}`. This dropdown detects which login methods have been added to the app and displays the appropriate controls. In our case, the only enabled login method is`accounts-password`, so the dropdown displays a password field. If you are adventurous, you can add the`accounts-facebook`package to enable Facebook login in your app - the Facebook button will automatically appear in the dropdown.

### Getting information about the logged-in user {#gettinginformationabouttheloggedinuser}

In your HTML, you can use the built-in`{{currentUser}}`helper to check if a user is logged in and get information about them. For example,`{{currentUser.username}}`will display the logged in user's username.

In your JavaScript code, you can use`Meteor.userId()`to get the current user's`_id`, or`Meteor.user()`to get the whole user document.

In the next step, we will learn how to make our app more secure by doing all of our data validation on the server instead of the client.

