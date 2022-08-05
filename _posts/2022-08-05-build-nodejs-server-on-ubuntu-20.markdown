---
layout: post
title: "Build Nodejs server on Ubuntu 20"
---


## 第一步：基礎建設

1. 在Linode開好node、build好Image（選擇Ubuntu 20.04）、設定好root密碼
2. 開啟終端機，輸入 `ssh root@xxx.xxx.xxx.xxx` ，輸入root密碼
3. `apt-get update`
4. 到 Linode Longview tab 開新的 longview，並複製安裝指令，例如：`curl -s https://lv.linode.com/B18F1605-C93A-A1A0-ED1A0591DDFD320D | sudo bash`
5. 安裝完成後執行指令 `sudo service longview restart`


## 第二步：Ubuntu設定

```bash
echo "YOUR_HOSTNAME" > /etc/hostname
hostname -F /etc/hostname
nano /etc/hosts
```

1. 設定時區，指令`dpkg-reconfigure tzdata`，依照提示選擇；完成後用指令 date 確認是否正確
2. 新增使用者
   1. 輸入指令`sudo adduser your_username`，新增使用者
   2. 接下來會提示輸入密碼及確認密碼
   3. 剩餘的資訊跳過省略不填寫
   4. 將使用者設定為超級管理員`sudo adduser your_username sudo`
   5. 完成後登出重新以該使用者登入
3. 更改ssh port
   1. `nano /etc/ssh/sshd_config`
   2. 找到 Port 22 將數值更改
   3. 重啟ssh，指令`sudo service ssh restart`
4. 更改為用key登入
   1. 在本地端產生key，使用指令`ssh-keygen` ，然後將`id_rsa.pub`的內容複製
   2. 在server新增`~/.ssh`資料夾，指令`mkdir ~/.ssh`
   3. 在`~/.ssh`下新增並編輯檔案`authorized_keys`
   4. 將複製的key內容貼上，儲存
   5. 編輯`/etc/ssh/sshd_config`，找到：
      ```
      PermitRootLogin yes
      PasswordAuthentication yes
      ```
      都改為`no`
   6. 重啟ssh
      ```
      sudo service ssh reload
      sudo service ssh restart
      ```
5. 建立防火牆
  1. 編輯`/etc/iptables.firewall.rules`
  2. 貼上以下內容並儲存（記得要更改中間的PORT為正確數值）
     ```
     *filter

     # Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
     -A INPUT -i lo -j ACCEPT
     -A INPUT -d 127.0.0.0/8 -j REJECT

     # Accept all established inbound connections
     -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

     # Allow all outbound traffic - you can modify this to only allow certain traffic
     -A OUTPUT -j ACCEPT

     # Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
     -A INPUT -p tcp --dport 80 -j ACCEPT
     -A INPUT -p tcp --dport 443 -j ACCEPT

     #  Allow SSH connections
     #  The -dport number should be the same port number you set in sshd_config
     -A INPUT -p tcp -m state --state NEW --dport <PORT> -j ACCEPT

     #  Allow ping
     -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

     #  Log iptables denied calls
     -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

     #  Allow incoming Longview connections
     -A INPUT -s longview.linode.com -j ACCEPT

     # Allow metrics to be provided Longview
     -A OUTPUT -d longview.linode.com -j ACCEPT

     #  Drop all other inbound - default deny unless explicitly allowed policy
     -A INPUT -j DROP
     -A FORWARD -j DROP

     COMMIT
     ```
     啟用該防火牆規則，指令`sudo iptables-restore < /etc/iptables.firewall.rules`

     檢查是否成功啟用`sudo iptables -L`

     編輯`/etc/network/if-pre-up.d/firewall`

     貼上以下內容並儲存
     ```shell
     #!/bin/sh
     /sbin/iptables-restore < /etc/iptables.firewall.rules
     ```

     更改檔案權限`sudo chmod +x /etc/network/if-pre-up.d/firewall`
6. 安裝 File2Ban
   ```
   sudo apt-get install fail2ban
   ```


## 第三步：安裝Nginx

```bash
# package update
sudo apt-get update

# nginx
sudo apt-get -y install nginx nginx-extras
```


## 第四步：安裝Node.js

```bash
sudo apt install nodejs

# 確認安裝成功
node -v

# 確認npm也安裝成功
npm -v
```

## 第五步：建立測試js

建立 `hello.js`

```bash
cd ~
nano hello.js
```

將以下內容貼上：

```javascript
const http = require('http');

const hostname = 'localhost';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World!\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

測試執行：

```bash
node hello.js
```

這時候另開一個視窗輸入下列指令：

```bash
curl http://localhost:3000
```

會得到以下回傳：

```
Hello World!
```


## 第六步：安裝PM2

**以下所提及的username，都請自動更改為您前面所自行設定的用戶名**

PM2是一個Node.js的程序管理程式，會維護並讓網站或程序在後台自動運行。

參照：[How To Set Up a Node.js Application for Production on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-20-04)

```bash
sudo npm install pm2@latest -g
```

安裝完成後可以使用以下指令跑`hello.js`：
```bash
pm2 start hello.js
```

會顯示類似以下狀態：
```bash
...
[PM2] Spawning PM2 daemon with pm2_home=/home/username/.pm2
[PM2] PM2 Successfully daemonized
[PM2] Starting /home/username/hello.js in fork_mode (1 instance)
[PM2] Done.
┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
│ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
│ 0  │ hello              │ fork     │ 0    │ online    │ 0%       │ 25.2mb   │
└────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘
```

如果應用程序crash或被kill，使用PM2執行的會自動將之重新啟動。我們執行以下指令讓PM2可以隨著系統開機自動啟動：

```bash
pm2 startup systemd
```

執行後會出現類似下方指示：

```bash
[PM2] Init System found: systemd
sammy
[PM2] To setup the Startup Script, copy/paste the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u username --hp /home/sammy
```

請複製並執行`sudo env PATH=$PATH:/usr/bin ...`這段

接著執行`pm2 save`來保存現有執行的程序

Start the service with `systemctl`:

```bash
sudo systemctl start pm2-username
```

Check the status of the systemd unit:

```bash
systemctl status pm2-username
```

其他：

```bash
# 列出 pm2 程序，你可以在這裡找到程序ID
pm2 list

# 停止 pm2 程序
pm2 stop app_name_or_id

# 重啟 pm2 程序
pm2 restart app_name_or_id
```


## 第七步：Nginx設定

參考：
- [How to Run Multiple Node.JS Applications on Ubuntu 20.04 with Nginx](https://www.vultr.com/docs/how-to-run-multiple-node-js-applications-on-ubuntu-20-04-with-nginx/)
- [How To Secure Nginx with Let's Encrypt on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)

```bash
# 開啟ufw防火牆
sudo ufw enable

# 檢查防火牆狀態
sudo ufw status

# 在防火牆中允許Nginx
sudo ufw allow 'Nginx Full'
# or
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
```

建立Nginx網站設定
```bash
sudo nano /etc/nginx/sites-available/example.com
```

貼入以下內容，domain和port部分請自行依需求更改
```
server {
  listen 80;
  listen [::]:80;

  server_name example.com www.example.com;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
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

網站或應用檔案可以放在`/var/www`下，如果網站有package.json記得要到目錄下`npm install`，否則會502錯誤
