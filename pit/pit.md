# Box 



![](img/Pasted%20image%2020210809074034.png)

https://www.hackthebox.eu/home/machines/profile/346

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.241 10.10.10.241
```

|Username | Passoword |
|-----------|------------|
| michelle | ied^ieY6xoquu|