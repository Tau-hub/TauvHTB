# Box 



![image-20210325154116318](img/image-20210325154116318.png)

https://www.hackthebox.eu/home/machines/profile/2

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#Reconnaissance)
* [Exploitation](#exploitation)
  + [Root](#root)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.4 10.10.10.4
```

![image-20210325154752765](img/image-20210325154752765.png)

After researching a bit, I found an exploit using smb1 on windows xp

## Exploitation

### Root

https://www.exploit-db.com/exploits/43970

After using it we are directly NT AUTHROITY\SYSTEM 

![image-20210325162116713](img/image-20210325162116713.png)