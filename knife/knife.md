# Box 



![image-20210802110438092](img/image-20210802110438092.png)

https://www.hackthebox.eu/home/machines/profile/347

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#reconnaissance)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [Root](#root)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.242 10.10.10.242
```

![image-20210802111305081](img/image-20210802111305081.png)

We have a website : 

![image-20210802112234504](img/image-20210802112234504.png)

Nothing particular, so I launch burp and check the headers. 



![image-20210802112308350](img/image-20210802112308350.png)

Here we can see something weird. We have a version of PHP `8.1.0-dev` . Let's take a look on searchploit. 

![image-20210802112351065](img/image-20210802112351065.png)

It seems it is a vulnerable version. Let's grab the script and try it. (https://www.exploit-db.com/exploits/49933)

## Exploitation

We have a shell.

![image-20210802112606243](img/image-20210802112606243.png)

## Post-Exploitation

### User

It seems that we are the user but we do not have a proper shell.

So let's try to get out user ssh key.

![image-20210802113127956](img/image-20210802113127956.png)

I couldn't log in in ssh because it requires a password. So let's grab the user.txt :

![image-20210802120253109](img/image-20210802120253109.png)

### Root

Our user james can execute `/usr/bin/knife` as sudo : 

![image-20210802120643527](img/image-20210802120643527.png)

Searching for this binary we can execute ruby code to do it. 

![image-20210802120844490](img/image-20210802120844490.png)

Let's grab a reverse shell and paste it. 

I used this one :  https://github.com/secjohn/ruby-shells/blob/master/revshell.rb

Upload your reverse shell in the machine.

![image-20210802121104748](img/image-20210802121104748.png)

![image-20210802121124933](img/image-20210802121124933.png)

Let's get our flag.

![image-20210802121147654](img/image-20210802121147654.png)

Rooted.

