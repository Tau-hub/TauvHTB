# Box 



![image-20210325150521600](img/image-20210325150521600.png)

https://www.hackthebox.eu/home/machines/profile/1

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Enumeration](#enumeration)
* [Exploitation](#exploitation)
  + [Root](#root)

# Contents 

## Enumeration

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.3 10.10.10.3
```

![image-20210325150722957](img/image-20210325150722957.png)

This box is really easy I've done it just for fun.

At first I tried the exploit the ftpd version but it didn't work.

https://www.exploit-db.com/exploits/17491

## Exploitation

### Root

By using searchsploit you can find an exploit on the SMB server.

![image-20210325153540016](img/image-20210325153540016.png)



Using metasploit to get our RCE we get a shell to root : 

![image-20210325153647202](img/image-20210325153647202.png)