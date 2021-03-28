# Box 



![image-20210326150440328](img/image-20210326150440328.png)

https://www.hackthebox.eu/home/machines/profile/121

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#Reconnaissance)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [Root](#root)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.75 10.10.10.75
```

![image-20210326150244258](img/image-20210326150244258.png)

So we have a website : 

![image-20210326152334058](img/image-20210326152334058.png)

Let's check the source code :

![image-20210326152424934](img/image-20210326152424934.png)

We now go the website : 

![image-20210326152657418](img/image-20210326152657418.png)

There is nothing interesting. Let's do some directory listing : 

```bash
ffuf -u http://10.10.10.75/nibbleblog/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt 
```



![image-20210326152805685](img/image-20210326152805685.png)



Let's read the README. 

![image-20210326152841635](img/image-20210326152841635.png)

We find an exploit with this specific version but we need admin's credential. 

https://www.exploit-db.com/exploits/38489

## Exploitation

After trying to login too much on the admin.php page, I got banned.

I think bruteforcing is not the way to go.

After checking all the files I foud nothing... So I got back and tried a few password.

```bash
admin:nibblesblog
admin:nibbles
```

and admin:nibbles worked. We  can now use our exploit. 

Start metasploit and use :

```
use exploit/multi/http/nibbleblog_file_upload
```

you should get your shell : 

![image-20210326155022932](img/image-20210326155022932.png)

## Post-Exploitation

### User

We already are the user. 

We can get our flag.

![image-20210326155338357](img/image-20210326155338357.png)

### Root

Let's do the classical  `sudo -l` :

![image-20210326155416629](img/image-20210326155416629.png)

There is no script withe this path but we have a personal.zip folder that we can unzip.

![image-20210326155528832](img/image-20210326155528832.png)

We now have full access to the script. Just edit it and remove the content. 

Add `/bin/bash` to the script and you are root.

![image-20210326160527823](img/image-20210326160527823.png)

Rooted.