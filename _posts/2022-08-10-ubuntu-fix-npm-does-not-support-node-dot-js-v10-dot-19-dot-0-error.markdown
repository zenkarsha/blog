---
layout: post
title: "Ubuntu fix npm does not support Node.js v10.19.0 error"
---

在升級了npm之後突然出現這個報錯，搜尋了一下紀錄以下解決方法：

```bash
curl -fsSL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

升級之後確認：
```bash
node -v
```

目前npm版本是`8.16.0`，node.js版本則是`v12.22.12`，可以正常使用npm

參考：[npm does not support Node.js v10.19.0](https://askubuntu.com/questions/1382565/npm-does-not-support-node-js-v10-19-0)
