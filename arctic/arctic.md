# Box 



[image]

https://app.hackthebox.eu/machines/Arctic

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.11 10.10.10.11
```

![](img/Pasted%20image%2020210820093729.png)

From what I see we only have RPC available and a port 8500 but we don't know what is behind.

Let's enumerate RPC then.