---
layout: post
title: "Fail2Ban ddos setting"
---

### How to use fail2ban to protect Apache

As you can see, this method will work for any server you have in front of your real web server, or to the actual web server itself, actually this will mainly protect your port 80.

Consider that you will have to adjust the path to your web server, I'll use varnish in my case.

Edit your `/etc/fail2ban/jail.local` file and add this section:

```
[http-get-dos]
enabled = true
port = http,https
filter = http-get-dos
logpath = /var/log/apache2/access.log
maxretry = 300
findtime = 300
#ban for 5 minutes
bantime = 600
action = iptables[name=HTTP, port=http, protocol=tcp]
```

Now we need to create the filter, to do that, create the file `/etc/fail2ban/filter.d/http-get-dos.conf` and copy the text below in it:

```
# Fail2Ban configuration file
#
# Author: http://www.go2linux.org
#
[Definition]

# Option: failregex
# Note: This regex will match any GET entry in your logs, so basically all valid and not valid entries are a match.
# You should set up in the jail.conf file, the maxretry and findtime carefully in order to avoid false positives.

failregex = <HOST> - - [[][^]]+[]] "(GET|POST) / HTTP/*

# Option: ignoreregex
# Notes.: regex to ignore. If this regex matches, the line is ignored.
# Values: TEXT
#
ignoreregex =
```

### Note

Be sure to adjust maxretry and findtime to some values that fits your needs.

- maxretry Is the maximum times of tries before the originating IP gets blocked.
- findtiem Is the time window (in seconds) where the maxretry times should occur, for the IP to get blocked.

As you can see in my example, I have set up 300 maxretry and 300 for findtime, so, we need to have 300 GETs from the same IP in a time window of 300 seconds to have the originating IP blocked.

Consider that you will have one GET for each css, js, html, ico and other files that are part of your webpage, so if you have 20 components, some client needs only to load 15 pages in 5 minutes to get blocked. Be sure to adjust those values to fit your needs.

### Test failregex

```
fail2ban-regex /var/log/apache2/access.log /etc/fail2ban/filter.d/http-get-dos.conf
```

如果有出現 match 即表示 failregex 沒問題，如沒有則需要調整 failregex

### Test `http-get-dos`

```
ab -n 1000 -c 2 http://www.domaini.com/
```

如果在超過 maxretry 後有產生 request fail，或是至 `/var/log/fail2ban.log` 看電腦的 ip 有沒有被 ban
