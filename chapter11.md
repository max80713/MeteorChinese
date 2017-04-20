# 測試\(Testing\) {#testing}

Now that we've created a few features for our application, let's add a test to ensure that we don't regress and that it works the way we expect.

We'll write a test that exercises one of our Methods \(which form the "write" part of our app's API\), and verifies it works correctly.

To do so, we'll add a[test driver](http://guide.meteor.com/testing.html#test-driver)for the[Mocha](https://mochajs.org/)JavaScript test framework:

```
meteor add practicalmeteor:mocha
```

We can now run our app in "test mode" by calling out a special command and specifying to use the driver \(you'll need to stop the regular app from running, or specify an alternate port with`--port XYZ`\):

```
meteor test --driver-package practicalmeteor:mocha
```

If you do so, you should see an empty test results page in your browser window.

Let's add a simple test \(that doesn't do anything yet\):

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

In any test we need to ensure the database is in the state we expect before beginning. We can use Mocha's`beforeEach`construct to do that easily:

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

Here we create a single task that's associated with a random`userId`that'll be different for each test run.

Now we can write the test to call the`tasks.remove`method "as" that user and verify the task is deleted:

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

There's a lot more you can do in a Meteor test! You can read more about it in the Meteor Guide[article on testing](http://guide.meteor.com/testing.html).

