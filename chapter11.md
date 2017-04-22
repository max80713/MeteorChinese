# 測試\(Testing\) {#testing}

在前幾個章節，我們加入了一些新功能。接下來，我們要來寫一些測試以確保程式會如我們預期的運作。

首先先寫一個測試，來測試寫入資料庫的功能，確保功能運作正常。

在這之前，我們先安裝[Mocha](https://mochajs.org/) JavaScript測試框架的驅動套件:

```
meteor add practicalmeteor:mocha
```

我們可以在測試模式\(test mode\)下執行我們的App並且指定我們要使用的驅動套件\(注意:必須先停止本來正在執行的App或是透過`--port XYZ`來指定另一個埠號，只要在終端機\(terminal\)輸入以下指令:

```
meteor test --driver-package practicalmeteor:mocha
```

你就會在瀏覽器上看到空白的測試結果頁。

現在我們來寫一個簡單的測試\(什麼都不做的測試\):

[imports/api/tasks.tests.js»](https://github.com/meteor/simple-todos/commit/92f2ca1d2865a5fd196879bb0185fd2edf3c619c)

```
/* eslint-env mocha */

import { Meteor } from 'meteor/meteor';

if (Meteor.isServer) {
  describe('Tasks', () => {
    describe('methods', () => {
      it('can delete owned task', () => {
      });
    });
  });
}
```

在每個測試開始之前，我們要確保資料庫中有我們要測試的資料，我們可以透過Mocha的`beforeEach`來存入資料:

[imports/api/tasks.tests.js»](https://github.com/meteor/simple-todos/commit/cd403a50cacba784de11a7a94e6d55bc884b33fb)

```
/* eslint-env mocha */

import { Meteor } from 'meteor/meteor';
import { Random } from 'meteor/random';
import { Tasks } from './tasks.js';

if (Meteor.isServer) {
  describe('Tasks', () => {
    describe('methods', () => {
      const userId = Random.id();
      let taskId;
      beforeEach(() => {
        Tasks.remove({});
        taskId = Tasks.insert({
          text: 'test task',
          createdAt: new Date(),
          owner: userId,
          username: 'tmeasday',
        });
      });
      it('can delete owned task', () => {
      });
    });
```

在這裡建立了一個隨機的`userId`，並且在測試開始之前透過這個`userId`來存入一個新的待辦事項到資料庫中。

然後我們再透過這個`userId`來測試`tasks.remove`的功能，確保該使用者可以刪除剛剛所存入的待辦事項:

[imports/api/tasks.tests.js»](https://github.com/meteor/simple-todos/commit/9a08b96bae018a4ecb3d23dada624accdb817cb0)

    import { Meteor } from 'meteor/meteor';
    import { Random } from 'meteor/random';
    import { assert } from 'meteor/practicalmeteor:chai';
    import { Tasks } from './tasks.js';

    ...some lines skipped...
          });
          it('can delete owned task', () => {
            // Find the internal implementation of the task method so we can
            // test it in isolation
            const deleteTask = Meteor.server.method_handlers['tasks.remove'];

            // Set up a fake method invocation that looks like what the method expects
            const invocation = { userId };

            // Run the method with `this` set to the fake invocation
            deleteTask.apply(invocation, [taskId]);

            // Verify that the method does what we expected
            assert.equal(Tasks.find().count(), 0);
          });
        });
      });

我們還有很多測式要寫!如果想深入了解Meteor的測試功能，可以參考[article on testing](http://guide.meteor.com/testing.html)。

