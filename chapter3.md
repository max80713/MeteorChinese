# 用模板\(template\)創造畫面\(view\) {#definingviewswithtemplates}

現在讓我們來修改一下上一章自動產出的程式碼，修改完之後再來談談細節。

首先，我們先把html檔的`<body>`移除，只留下`<head>`標籤:

```
<head>
  <title>simple</title>
</head>
```

再來我們新建一個資料夾`imports`，並且在這個資料夾底下再建立一個資料夾`ui`，然後在這個資料夾底下新增一個檔案`body.html`:

```
<body>
  <div class="container">
    <header>
      <h1>Todo List</h1>
    </header>
    <ul>
      {{#each tasks}}
        {{> task}}
      {{/each}}
    </ul>
  </div>
</body>

<template name="task">
  <li>{{text}}</li>
</template>
```

以及`body.js`:

```
import { Template } from 'meteor/templating';
import './body.html';
Template.body.helpers({
  tasks: [
    { text: 'This is task 1' },
    { text: 'This is task 2' },
    { text: 'This is task 3' },
  ],
});
```

資料夾`imports`底下的檔案必須載入\(import\)才會在啟動App的時候被執行，所以我們需要在前端網頁的進入點`client/main.js`

載入它們，並且刪掉裡面本來的程式碼:

```
import '../imports/ui/body.js';
```

如果你想了解`import`的作用或是如何管理程式碼的結構，可以參考Meteor指南裡的[Application Structure article](http://guide.meteor.com/structure.html)。

現在在瀏覽器上，你的App大致上看起來會是這樣子:

> #### Todo List {#todolist}
>
> * This is task 1
> * This is task 2
> * This is task 3

我們來解釋一下目前這些程式碼的功能。

### Meteor中的HTML檔案負責建立模板\(template\) {#htmlfilesinmeteordefinetemplates}

Meteor解析HTML檔案時，只認得三大主要標籤`<head>`、`<body>`、`<template>`。所有放在`<template>`標籤裡頭的程式碼都會被編譯成Meteor的模板\(template\)，這些模板可以在HTML檔裡透過`{{>templateName}}`或是在JS檔裡透過`Template.templateName`的方式引用。而`<body>`則可以透過 `Template.body`來引用，可以把它想最上層的模板\(template\)，能夠包住任何其他的模板。

### Adding logic and data to templates {#addinglogicanddatatotemplates}

All of the code in your HTML files is compiled with[Meteor's Spacebars compiler](https://github.com/meteor/meteor/blob/devel/packages/spacebars/README.md). Spacebars uses statements surrounded by double curly braces such as`{{#each}}`and`{{#if}}`to let you add logic and data to your views.

You can pass data into templates from your JavaScript code by defining_helpers_. In the code above, we defined a helper called`tasks`on`Template.body`that returns an array. Inside the body tag of the HTML, we can use`{{#each tasks}}`to iterate over the array and insert a`task`template for each value. Inside the`#each`block, we can display the`text`property of each array item using`{{text}}`.

In the next step, we will see how we can use helpers to make our templates display dynamic data from a database collection.

