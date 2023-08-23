---
layout: post
title: "Temp fix old ssh-rsa issue on macOS Ventura"
---


macOS 13 (Ventura) ships with OpenSSH_9.0p1. According to the OpenSSH release notes:

> This release disables RSA signatures using the SHA-1 hash algorithm by default. This change has been made as the SHA-1 hash algorithm is cryptographically broken, and it is possible to create chosen-prefix hash collisions for USD$50K ...

因為 `SHA-1 hash algorithm` 安全性問題，macOS 不再支援該加密方式，所以舊有的 ssh 如果是用 keypass 方式連線都會出現 `Permission denied (publickey)` 錯誤而無法連線

---

## 修復方法 temp fix method

open folder `/etc/ssh`

if you can not see the hidden folders, press `command + shift + .`

add following lines to the end of file `ssh_config` (after `Host *` ..., remember to add tabs):

```
HostkeyAlgorithms +ssh-rsa
PubkeyAcceptedAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
```

then save the file and reboot your mac.


## 系統版本 macOS version

my current macOS version is: 13.4.1 (c) (22F770820d).


## 參考網站 reference

- [SSH Key Authentication Fails with macOS Ventura #1003](https://github.com/sshnet/SSH.NET/issues/1003)
- [Git SSH "permission denied" in macOS 13 Ventura](https://superuser.com/questions/1749364/git-ssh-permission-denied-in-macos-13-ventura)
- [Visual Studio 2022 won't connect via SSH on macOS after upgrading to Ventura](https://stackoverflow.com/questions/74215881/visual-studio-2022-wont-connect-via-ssh-on-macos-after-upgrading-to-ventura)
