# Box 



![](img/box_image.png)

https://www.hackthebox.eu/home/machines/profile/356

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.247 10.10.10.247
```

![](img/Pasted%20image%2020210806085007.png)

Getthing here I found an exploit for freeCiv here : https://packetstormsecurity.com/files/163311/Android-2.0-FreeCIV-Arbitrary-Code-Execution.html

## Exploitation

Reading the script it is obvious that it will be used later since I need to use ssh to tunnel a port. 

However we need more information on the box so I did a new nmap with every ports. 

Here is the result : 

![](img/Pasted%20image%2020210806091939.png)

The first weird thing for me was the ES file Explorer http server. 
Looking for an exploit I found a python script to read files on the device : 

https://www.exploit-db.com/exploits/50070 

We can use this tool to list all files, I found a file called `creds.jpg` : 

```bash
python3 50070.py listPics 10.10.10.247
```

![](img/Pasted%20image%2020210806093934.png)

Let's download the image

![](img/Pasted%20image%2020210806094036.png)

That's what we get:  

![](img/Pasted%20image%2020210806085042.png)


| Username | Password |
|----------|--------|
| kristi| Kr1sT!5h@Rp3xPl0r3!|

## Post-Exploitation
### User
Using the login we have a shell and can get the user.txt file
![](img/Pasted%20image%2020210806094329.png)
![](img/Pasted%20image%2020210806094353.png)

### Root
Remember what we found at first? Let's use it now. 
https://packetstormsecurity.com/files/163311/Android-2.0-FreeCIV-Arbitrary-Code-Execution.html

Using this script will create a tunnel redirecting all the traffic in the port 5555 to your localhost. 

Now you can use adb  we are going to use adb to get a shell.
```bash
adb connect localhost:5555
```

![](img/Pasted%20image%2020210806094919.png)

then  `adb shell`
![](img/Pasted%20image%2020210806094956.png)

`su -` and use `find / -name "root.txt"` to get your `root.txt` file.

![](img/Pasted%20image%2020210806095102.png)

Rooted.