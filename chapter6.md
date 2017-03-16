# 在Android或是iOS上執行你的App {#runningyourapponandroidorios}

> 目前Meteor還不支援在Windows上產生可執行的App。如果你正在Windows上進行本教學，可以跳過此章節。

到目前為止，我們建立了我們的App並且在瀏覽器上執行，但Meteor本身為跨平台運行所設計 - 只要下幾個簡單的指令，你的待辦事項App網站就可以變成在iOS或是Android上執行的App。

Meteor已經替我們準備好所有產生手機App所需要的工具，但下載這些工具會花一點時間 - 產生Android App的工具大約300MB左右，而產生iOS App則需先自行安裝大約2GB的Xcode。

### 在iOS模擬器上執行App \(限Mac電腦\) {#runningonaniossimulatormaconly}

如果你使用Mac電腦，你可以在iOS模擬器上執行你的App。

在終端機進入到你的App資料夾中，然後輸入:

```
meteor install-sdk ios
```

便會開始自動建立iOS App。完成之後再輸入:

```
meteor add-platform ios
meteor run ios
```

你就會看到你的App在iOS模擬器中執行囉!

### 在Android模擬器上執行App \(限Mac電腦\) {#runningonanandroidemulator}

在終端機進入到你的App資料夾中，然後輸入:

```
meteor install-sdk android
```

便會開始自動下載所以需要的工具並且建立Android App。完成之後再輸入:

```
meteor add-platform android
meteor run android
```

初始化之後，你就會看到你的App在Android模擬器中執行囉!在Android模擬器中執行App速度會比較慢，如果你想測試實際的情況，應該在真正的手機上執行。

### 在Android裝置上執行App {#runningonanandroidemulator}

首先，完成上述步驟以在你的電腦上設置好建立Android App所需要的工具。然後確認你開啟手機上[USB Debugging](http://developer.android.com/tools/device.html#developer-device-options)的功能並且將手機透過USB連接至電腦。

在終端機輸入:

```
meteor run android-device
```

便會自動建立並且安裝App到你的手機上。

### 在iPhone或iPad上執行App \(限Mac，且需Apple開發者帳戶\) {#runningonaniphoneoripadmaconlyrequiresappledeveloperaccount}

如果你有Apple開發者帳戶\(developer account\)，你可以在iOS上裝置執行你的App。在終端機輸入:

```
meteor run ios-device
```

便會在Xcode中自動為你的iOS App建立一個新的專案。你可以透過Xcode來在任何Xcode支援的裝置上執行你的App。

我們已經了解到在手機上執行你的App是多麼簡單的一件事了!接下來我們繼續新增更多的功能到我們的App吧!

