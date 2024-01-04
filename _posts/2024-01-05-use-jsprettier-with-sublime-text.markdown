---
layout: post
title: "在Sublime Text裡使用JsPrettier自動整理程式碼（for typescript）"
---

因為要在Sublime Text寫Typescript，看到youtube上的教學感覺Visual Studio Code可以用Prettier直接存檔把格式整理好很方便，但是基於信仰不願意跳槽到VSC（雖然一直耳聞真的很好用），但是用了多年的Sublime Text，已經習慣大部分的操作。

這次會用到JsPrettier是因為在學習Typescript，原本使用的套件不支援.ts。

首先用install package找到JsPrettier package，然後按下enter安裝，之後到設定裡把`auto_format_on_save`改為`true`，應該就大功告成了！

一切看起來平安無事，但是在.ts的檔隨意改了一下發現沒有作用，接著就是不斷爬文試錯的開始，原本在.js檔案是可以正常使用的，但是中間想說去改改看package的程式碼之後，整個套件就完全不能運作了⋯⋯

查看了一下console的debug訊息：

```
[JsPrettier DEBUG]: Changing working directory to '/Users/USERNAME/Desktop/hello-world'
[JsPrettier DEBUG]: Could not resolve Prettier config file, will use options defined in Sublime Text.
[JsPrettier ERROR]: Ensure Prettier is installed and defined in your environment PATH variable. You can optionally specify a custom path in 'JsPrettier.sublime-settings' using the 'prettier_cli_path' setting.
```

看起來是CLI壞掉了，嘗試用文件中的幾個範例測試都還是沒辦法使用，remove套件之後重新安裝了兩三次也都無效。在官方GitHub上也沒有查到類似案例，這種問題問ChatGPT大概也只會得到官腔式的回答。之後在原本以為即將被ChatGPT打敗的stackoverflow找到了解方：

關鍵是要到`prettier_cli_path`的位置，但是找不到，所以要改方式直接用`npm`安裝：

```
npm install --global prettier
```

之後查詢路徑

```
which prettier
```

我得到的路徑是`/opt/homebrew/bin/prettier`，將這段路徑貼到Settings > PackageSettings > JsPrettier > Settings-Default （或是Setting-User）

```
{
  ...

  "prettier_cli_path": "/opt/homebrew/bin/prettier",

  ...
}
```

之後就可以正常使用了，繼續來學Typescript⋯⋯

---

### 參考資料
- [https://stackoverflow.com/questions/46191137/how-to-properly-re-configure-jsprettier-after-an-update-with-a-breaking-change](https://stackoverflow.com/questions/46191137/how-to-properly-re-configure-jsprettier-after-an-update-with-a-breaking-change)
- [https://github.com/jonlabelle/SublimeJsPrettier](https://github.com/jonlabelle/SublimeJsPrettier)
