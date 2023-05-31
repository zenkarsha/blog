---
layout: post
title: "Run simple HTML website on Ubuntu 20"
---


## 前置設定

參考：
- [Build Nodejs server on Ubuntu 20](https://zenkarsha.github.io/blog/2022/08/05/build-nodejs-server-on-ubuntu-20)


## 設置網站

建立網站資料夾
```bash
sudo mkdir /var/www/example.com
```

更改資料夾所有人
```bash
sudo chmod username:username /var/www/example.com
```

建立Nginx網站設定
```bash
sudo nano /etc/nginx/sites-available/example.com
```

貼入以下內容，domain部分請自行依需求更改
```
server {
  listen 80;
  listen [::]:80;

  server_name example.com;

  root /var/www/example.com;
  index index.html;

  location / {
    try_files $uri $uri/ =404;
  }
}
```

確認設定正常：`sudo nginx -t`

將設定檔複製到`sites-enabled`
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled
```

重啟Nginx

```bash
sudo service nginx restart
# or
sudo systemctl restart nginx
```

接著只要把domain DNS設定好應該就可以正常連線了
