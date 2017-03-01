# 使用表單\(form\)新增待辦事項 {#addingtaskswithaform}

在這一章，我們將在頁面上加入一個輸入欄位來讓使用者透過這個欄位新增待辦事項。

首先，我們先新增一個表單到HTML檔:'

[imports/ui/body.html»](https://github.com/meteor/simple-todos/commit/06fc0de9e7665c11170f69f7df069229fa99330f)

```
<div class="container">
  <header>
    <h1>Todo List</h1>
    <form class="new-task">
      <input type="text" name="text" placeholder="Type to add new tasks" />
    </form>
  </header>
```

然後再加入以下的程式碼到JS檔，以監聽表單的`submit`事件\(event\):

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/2fd36714d6494f4fb1bf99d1aefbc9a10dfde350)

```
Template.body.events({
  'submit .new-task'(event) {
    // Prevent default browser form submit
    event.preventDefault();

    // Get value from form element
    const target = event.target;
    const text = target.text.value;

    // Insert a task into the collection
    Tasks.insert({
      text,
      createdAt: new Date(), // current time
    });

    // Clear form
    target.text.value = '';
  },
});
```

現在你的App有了一個新的輸入欄位，只要在這個欄位輸入文字然後按Enter，就會新增一個待辦事項。如果開啟一個新的瀏覽器視窗，然後再次打開App，你會發現新開啟的App也能看到剛才加入的待辦事項，這代表所有的使用者看到的待辦事項都同步了!

### 將事件\(event\)依附到模板\(template\) {#attachingeventstotemplates}

在模板中加入事件監聽器\(event listeners\)的方式和加入helper\(輔助器\)的方式一模一樣: 呼叫`Template.templateName.events(...)`

並且傳入字典\(dictionary\)型態的資料。字典的鍵\(key\)描述監聽的事件，字典的值為事件處理器\(event handler\)，當事件發生時便會呼叫它。

以上的範例中，我們利用CSS選擇器`.new-task`對每一個class為new-task的標籤元素監聽`submit`事件。當使用者在輸入欄位中按下Enter就會觸發這個事件，接著就會自動呼叫事件處理器。

事件處理器函數會接收一個叫做`event`的參數，這個參數包含被觸發事件的資訊。在以上的範例中，`event.target`指的是表單裡的輸入欄位，我們可以使用`event.target.text.value`抓到欄位裡所填入的值。你可以透過`console.log(event)`來取得`event`這個物件的所有屬性，並且顯示在瀏覽器的主控台中。

在事件處理器函數的最後一行，我們清除輸入欄位裡的值以便再次加入新的待辦事項。

### 將待辦事項存入集合\(collection\) {#insertingintoacollection}

在事件處理器函數中，我們透過呼叫`Tasks.insert()`來將新的待辦事項存入集合中。由於我們並不需要事先定義好集合的欄位結構\(schema\)，所以可以給task物件任何的屬性與對應值。

從瀏覽器可以存入任何的資料到資料庫裡是一件很不安全的事情，目前先假裝沒看到\(?，在第10章我們將會學到如何限制資料庫的存取以確保資料庫的安全性。

### 將待辦事項排序 {#sortingourtasks}

一般來說，使用者會希望最新的待辦事項可以顯示在清單的最上方，這正好和目前的情況相反: 我們的App會把最新的待辦事項顯示在清單的最下方。

我們只要在`tasks`輔助器\(helper\)中呼叫`find`時傳入用來排序的參數，針對`createdAt`來排序，就可以解決上述的問題囉!

[imports/ui/body.js»](https://github.com/meteor/simple-todos/commit/3706dd75816a2a0ac525aa79acdb75f622e64d5a)

```
Template.body.helpers({
  tasks() {
    // Show newest tasks at the top
    return Tasks.find({}, { sort: { createdAt: -1} });
  },
});
```

在下一章，我們將會在頁面上再加入待辦事項App兩個非常重要的功能:「完成」與「刪除」。

