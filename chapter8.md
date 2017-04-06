# 新增使用者帳戶 {#addinguseraccounts}

Metoer內建帳戶系統和登入系統使用者介面來讓你快速地完成會員系統的功能。

我們必須加入相關套件來開啟這些功能。在你的App資料夾底下，輸入:

```
meteor add accounts-ui accounts-password
```

在HTML檔裡頭，加入以下的程式碼來在頁面上新增一個登入的下拉式選單:

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/bc9fb936699c1ce8a0643e5c563043d560a04811)

```
        Hide Completed Tasks
      </label>

      {{> loginButtons}}

      <form class="new-task">
        <input type="text" name="text" placeholder="Type to add new tasks" />
      </form>
```

然後在JavaScript檔裡頭，加入以下的程式碼來修改帳戶系統使用者介面，把電子信箱改成使用者名稱:

[imports/startup/accounts-config.js»](https://github.com/meteor/simple-todos/commit/7c48c9aa89e26eac39cc67046f85e54bab5889fe)

```
import { Accounts } from 'meteor/accounts-base';
Accounts.ui.config({
  passwordSignupFields: 'USERNAME_ONLY',
});
```

我們還需要在客戶端的JavaScript程式進入點載入相關套件:

[client/main.js»](https://github.com/meteor/simple-todos/commit/47fde1a42d5d6d1b765b2f16d0f0cc48e0567be1)

```
import '../imports/startup/accounts-config.js';
import '../imports/ui/body.js';
```

現在使用者可以建立帳戶並且登入你的App了! 再讓我們增加兩項功能:

1. 只呈現新增待辦事項的輸入欄位給登入的使用者看到
2. 在每個待辦事項顯示是哪個使用者新增的

我們需要在`tasks`集合新增兩個欄位:

1. `owner` - 新增該待辦事項的使用者`_id`
2. `username` - 新增該待辦事項的使用者名稱 `username` 。 我們直接把使用者名稱存到待辦事項物件中，這樣我們就可以直接顯示用者名稱而不需要透過使用者 `_id`來查找使用者名稱。

首先，在`submit .new-task`事件處理器中加入以下的程式碼來儲存這兩個欄位:

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

再來，在HTML檔裡頭加入一個`#if`區塊以只有當使用者登入才呈現新增待辦事項的輸入欄位:

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

最後，加入一個Spacebars敘述來在每個待辦事項之後顯示使用者名稱:

[imports/ui/task.html»](https://github.com/meteor/simple-todos/commit/da75b1705c5d5ae3470f47406c261d4303f95a87)

```
    <input type="checkbox" checked="{{checked}}" class="toggle-checked" />

    <span class="text"><strong>{{username}}</strong> - {{text}}</span>
  </li>
</template>
```

現在，使用者可以登入並且知道每個待辦事項是被哪個使用者新增的了!讓我們來瞭解一下以上這些功能的細節吧!

### 自動化的帳戶系統使用者介面 {#automaticaccountsui}

如果你的App有`accounts-ui`套件，我們只要透過`{{>loginButtons}}` 幫助器\(helper\)來套入`loginButtons`模板就可以在頁面上建立一個登入的下拉式選單。這個下拉式選單會偵測App加入了那些登入方法並且顯示適當的控制項。在這個例子中，我們只加入了`accounts-password`這個登入方法套件，所以下拉式選單顯示了密碼輸入欄位。你可以試著加入其他的登入方法套件，例如:加入`accounts-facebook`這個套件可以使用臉書帳戶來登入你的App，下拉式選單便會顯示臉書的按鈕。

### 得到登入使用者資訊 {#gettinginformationabouttheloggedinuser}

在HTML檔裡頭，你可以使用內建的`{{currentUser}}`幫助器\(helper\)來檢查使用者是否登入並且取得登入使用者資訊。例如:透過`{{currentUser.username}}`取得並顯示登入使用者名稱。

在JavaScript程式碼中，你可以透過`Meteor.userId()`來取得目前使用者`_id` 或是`Meteor.user()`來取得整個使用者的文件\(document\)。

在下一章，我們將會學到如何在伺服器端驗證我們的資料來讓我們App更具安全性。

