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

### 在模板中加入資料 {#addinglogicanddatatotemplates}

所有在HTML檔裡的程式碼都會被[Meteor's Spacebars compiler](https://github.com/meteor/meteor/blob/devel/packages/spacebars/README.md)編譯。Spacebars可以識別被雙重大括號包起來的敘述，像是`{{#each}}`還有`{{#if}}`，讓你可以在App的畫面上呈現資料。而資料則是在JS檔裡透過helpers把傳到模板裡。

在上方的程式碼中，我們在`Template.body`中建立了一個叫做`tasks`的helper，這個helper傳送了一個資料陣列\(array\)到`<body>`標籤中。 所以我們可以在`<body>`裡透過`{{#each tasks}}`將陣列中的每一筆資料插入`task`模板，並且在`#each`的區塊裡，透過`{{text}}`顯示text屬性所對應的值。

在下一章，我們將會介紹如何使用helpers來將資料從資料庫中呈現到模板上。

