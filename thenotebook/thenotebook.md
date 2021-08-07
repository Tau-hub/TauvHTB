# Box 



![image-20210312152700744](img/image-20210312152700744.png)

https://www.hackthebox.eu/home/machines/profile/320

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.230 10.10.10.230
```

![image-20210312152910236](img/image-20210312152910236.png)

We got a webpage where we have to register and we can submit notes : 

![image-20210312154411846](img/image-20210312154411846.png)

After enumerating  I found the admin page where we can make notes 

![image-20210312154504756](img/image-20210312154504756.png)

I found nothing else so I took a look  on the differents headers and cookies we have : 

![image-20210312173710366](img/image-20210312173710366.png)



It seems that we have JWT token. Let's try to see if it's vulnerable.

![image-20210312174034598](img/image-20210312174034598.png)

There is a field `kid = "http://localhost:7070/privKey.key"`, let's try to setup a python server on our machine and send a request. 

![image-20210312174216200](img/image-20210312174216200.png)

Change the kid value : 

```
kid = "http://10.10.14.17:4444/"
```

Well, we have a request. Let's try to get a reverse_shell now

![image-20210312174230434](img/image-20210312174230434.png)

So we know we can modify this request to get to our own key.

We have to modify the `admin_cap` value to `1` and we have to modify the key used to verify the signature and generate our own pair of keys.

To generate a pair of key I used this link : https://gist.github.com/ygotthilf/baa58da5c3dd1f69fae9

## Exploitation

Then, you have to modify your JWT token (https://jwt.io/) like this : 

![firefox_QQG1JO45iz](img/firefox_QQG1JO45iz.png)

In the "verify signature" part you have to cat your keys and add the public and private key in plain text.

Then load your token in a burp request :

![image-20210312184045047](img/image-20210312184045047.png)

Send your request to the /admin and you should be fine ! 

![image-20210312184111267](img/image-20210312184111267.png)



Earlier during my Reconnaissance I found a /upload file where I can upload my reverse shell : 

![image-20210312184210520](img/image-20210312184210520.png)

After uploading my reverse shell, we got in : 

![image-20210312184518728](img/image-20210312184518728.png)

## Post-Exploitation

### User

After landing on the box, I did an Reconnaissance and found a file in `/var/backups` called `home.tar.gz`

![image-20210312190249034](img/image-20210312190249034.png)

We know have the private key and we can log in as noah.

![image-20210312190504572](img/image-20210312190504572.png)

### Root

Let's do the classical `sudo -l` :

![image-20210312190807124](img/image-20210312190807124.png)

Well, we have a docker exec in sudo, it must be the way.

After a while I found a PoC of a specific CVE https://github.com/0xBADCA7/CVE-2019-5736-PoC

Using this link and the tutorial I modified the go script to have another payload 

![image-20210312201839778](img/image-20210312201839778.png)

You start the exploit on your docker container : 

![image-20210312201903220](img/image-20210312201903220.png)



Execute your `/bin/sh` using your sudo docker exec:

![image-20210312201948803](img/image-20210312201948803.png)

You got your flag.

![image-20210312202006283](img/image-20210312202006283.png)

You can also modify the payload to have a reverse shell : 

![image-20210312202710074](img/image-20210312202710074.png)

Rooted.