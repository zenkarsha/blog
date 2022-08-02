---
layout: post
title: "Simple Nginx server setup"
---

## 第一步：基礎建設

1. 在Linode開好node、build好Image（選擇Ubuntu 14.04）、設定好root密碼
2. 開啟終端機，輸入 ssh root@xxx.xxx.xxx.xxx ，輸入root密碼
3. Ubuntu 16.04 issue (optional)
   ```
   open /etc/gai.conf
   uncomment precedence ::ffff:0:0/96 100 這行
   ```
   See: [apt-get update stuck: Connecting to security.ubuntu.com](https://askubuntu.com/questions/620317/apt-get-update-stuck-connecting-to-security-ubuntu-com)
4. apt-get update
5. 安裝curl `apt-get install curl`
6. 到 Linode Longview tab 開新的 longview，並複製安裝指令，例如：`curl -s https://lv.linode.com/B18F1605-C93A-A1A0-ED1A0591DDFD320D | sudo bash`
7. 安裝完成後執行指令 `sudo service longview restart`


## 第二步：Ubuntu設定

```
echo "YOUR_HOSTNAME" > /etc/hostname
hostname -F /etc/hostname
nano /etc/hosts
```

將檔案中的第二行：
```
127.0.1.1    ubuntu.members.linode.com    ubuntu
```
改為
```
127.0.1.1    your-hostname
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


## 第三步：安裝LNMP

```
# package update
sudo apt-get update

# nginx
sudo apt-get -y install nginx nginx-extras
```

```
# mysql
14.04
sudo apt-get -y install mysql-server
sudo mysql_install_db
sudo mysql_secure_installation

16.04
// https://askubuntu.com/questions/835541/mysql-install-db-on-ubuntu-16-04
mkdir -p /var/lib/mysql
sudo apt-get install mysql-server mysql-client mysql-common
chown -R mysql.mysql /var/lib/mysql
chmod -R 775 /var/lib/mysql

// If you don't want apparmor to make unnecessary issues, better disable it as follows.
sudo service apparmor stop
sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/
sudo service apparmor restart
sudo aa-status

sudo reboot
mysql_install_db
sudo mysql_secure_installation
```

```
# php 5
sudo apt-get -y install php5-cli php5-fpm php5-mysql php5-gd php5-curl

# php 7
sudo apt-get install software-properties-common python-software-properties
sudo apt-add-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install php7.0-cli php7.0-fpm php7.0-mysql
sudo apt-get install php7.0-mbstring php7.0-dom
sudo apt-get install php7.0-gd
```


## 第四步：安全性設定

php

```
# php5
sudo nano /etc/php5/fpm/php.ini

# php7
sudo nano /etc/php/7.0/fpm/php.ini
```

找到
```
;cgi.fix_pathinfo=1
```
改為
```
cgi.fix_pathinfo=0
```

### 重啟php

php5
```
sudo service php5-fpm restart
```

php7
```
sudo service php7.0-fpm restart
```

更改mysql port，編輯`/etc/mysql/my.cnf`
```
找到
[client]
port       = 3306
以及
[mysqld]
port       = 3306
更改port數值
```

重啟mysql`sudo service mysql restart`


## 第五步：設定Site config

`sudo nano /etc/nginx/sites-available/default`

將內容全數刪除並貼上以下內容
```
#php5
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www;
    index index.php index.html index.htm;
    server_name localhost;
    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_read_timeout 30;
        include fastcgi_params;
    }
}

#php7
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www;
    index index.php index.html index.htm;
    server_name localhost;
    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
    }
}
```

更改`nginx.conf`設定：`sudo nano /etc/nginx/nginx.conf`

將內容全數刪除並貼上以下內容
```
user www-data;
worker_processes 2;
worker_cpu_affinity 01 10;
worker_rlimit_nofile 512;
pid /run/nginx.pid;

events {
    use epoll;
    worker_connections 512;
}

http {
    proxy_buffer_size   128k;
    proxy_buffers   4 256k;
    proxy_busy_buffers_size   256k;

    client_max_body_size 20m;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;
    gzip_disable "msie6";

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

創建網站目錄資料夾`sudo mkdir /var/www`

新增並編輯檔案`~/add-domain.sh`

php5
```
#!/bin/bash

echo "Input Records (A/CNAME)";
read records;
echo "Records: $records";

if [ "${records}" == "A" ]; then

echo "Input domain name:";
read domainName;
echo "Domain: $domainName";

echo "Input root path:";
read rootPath;
echo "Root path: $rootPath";

cat <<EOT > $domainName
server{
        listen 80;
        server_name $domainName;
        root $rootPath;
        index index.php index.html index.htm;

        try_files \$uri \$uri/ @rewrite;

        location @rewrite {
                rewrite ^/(.*)$ /index.php?_url=/\$1;
        }

        location ~ \\.php$ {
                try_files $uri /index.php =404;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
                include fastcgi_params;
        }
}
EOT

#move to config
mv $domainName /etc/nginx/sites-available/$domainName

#create link to site-enable
ln -s /etc/nginx/sites-available/$domainName /etc/nginx/sites-enabled/$domainName

#reload and restart server
service nginx reload
service nginx restart

elif [ "${records}" == "CNAME" ]; then

echo "Input aliases to";
read aliases;
echo "aliases: $aliases";

fi
```

php7
```
#!/bin/bash

echo "Input Records (A/CNAME)";
read records;
echo "Records: $records";

if [ "${records}" == "A" ]; then

echo "Input domain name:";
read domainName;
echo "Domain: $domainName";

echo "Input root path:";
read rootPath;
echo "Root path: $rootPath";

cat <<EOT > $domainName
server{
        listen 80;
        server_name $domainName;
        root $rootPath;
        index index.php index.html index.htm;

        try_files \$uri \$uri/ @rewrite;

        location @rewrite {
                rewrite ^/(.*)$ /index.php?_url=/\$1;
        }

        location ~ \\.php$ {
                try_files $uri /index.php =404;
                fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
                include fastcgi_params;
        }
}
EOT

#move to config
mv $domainName /etc/nginx/sites-available/$domainName

#create link to site-enable
ln -s /etc/nginx/sites-available/$domainName /etc/nginx/sites-enabled/$domainName

#reload and restart server
service nginx reload
service nginx restart

elif [ "${records}" == "CNAME" ]; then

echo "Input aliases to";
read aliases;
echo "aliases: $aliases";

fi
```

更改檔案權限`sudo chmod +x add-domain.sh`

新增domain方法：`sudo ./add-domain.sh`


## 第六步：安裝Laravel環境

1. 安裝Composer
   ```
   sudo apt-get install git
   curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
   ```
2. `sudo apt install zip unzip php7.0-zip`
3. 安裝 laravel 5.3 需將 php 升級至 5.6.4 以上
   ```
   #php5
   sudo add-apt-repository ppa:ondrej/php
   sudo apt-get update
   sudo apt-get install php5.6

   # php7
   sudo apt-get install php7.0
   ```
