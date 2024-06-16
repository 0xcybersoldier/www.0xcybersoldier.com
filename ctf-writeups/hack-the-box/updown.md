# UpDown

<figure><img src="../../.gitbook/assets/UpDown.png" alt=""><figcaption></figcaption></figure>

UpDown is a medium difficulty Linux machine with SSH and Apache servers exposed. On the Apache server a web application is featured that allows users to check if a webpage is up. A directory named `.git` is identified on the server and can be downloaded to reveal the source code of the `dev` subdomain running on the target, which can only be accessed with a special `HTTP` header. Furthermore, the subdomain allows files to be uploaded, leading to remote code execution using the `phar://` PHP wrapper. The Pivot consists of injecting code into a `SUID` `Python` script and obtaining a shell as the `developer` user, who may run `easy_install` with `Sudo`, without a password. This can be leveraged by creating a malicious python script and running `easy_install` on it, as the elevated privileges are not dropped, allowing us to maintain access as `root`.

ip address : 10.10.11.177

lets start with nmap

```
nmap -A 10.10.11.177
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-15 09:08 +03
Stats: 0:00:23 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.90% done; ETC: 09:08 (0:00:00 remaining)
Stats: 0:00:33 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.95% done; ETC: 09:08 (0:00:00 remaining)
Nmap scan report for 10.10.11.177
Host is up (0.21s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Is my Website up ?
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.11 seconds
```

on main page we tested the website allow traffic out of node itself

<figure><img src="../../.gitbook/assets/1 (2).png" alt=""><figcaption><p>small server on 80 port</p></figcaption></figure>

next lets try find what is next find hidden subdomains

```
gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.11.177/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.177/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/dev                  (Status: 301) [Size: 310] [--> http://10.10.11.177/dev/]
```

its empty page

<figure><img src="../../.gitbook/assets/2 (2).png" alt=""><figcaption><p>dev dir </p></figcaption></figure>

so lets try find anther subdomains in under dev

```
cybersoldier@parrot]─[~]
└──╼ $gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://10.10.11.177/dev
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.177/dev
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.git                 (Status: 301) [Size: 315] [--> http://10.10.11.177/dev/.git/]
/.git/HEAD            (Status: 200) [Size: 21]
/.git/index           (Status: 200) [Size: 521]
/.git/logs/           (Status: 200) [Size: 1143]
/.git/config          (Status: 200) [Size: 298]
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
Progress: 583 / 4730 (12.33%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 591 / 4730 (12.49%)
===============================================================
Finished
===============================================================

```

its looks we can found .git folder lets try the following

<figure><img src="../../.gitbook/assets/3 (2).png" alt=""><figcaption><p>.git dir</p></figcaption></figure>

**GitTools**:

> GitTools is a collection of Python scripts designed to download and extract a git repository from a website.

```
git clone https://github.com/internetwache/GitTools.git
cd GitTools
```

> **Dumper**: To download the `.git` folder.

```
cd Dumper
./gitdumper.sh http://target.com/.git/ /path/to/download/
```

```
cybersoldier@parrot]─[~/Desktop/updown]
└──╼ $../tools/GitTools/Dumper/gitdumper.sh http://10.10.11.177/dev/.git/ .
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating ./.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[-] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[+] Downloaded: packed-refs
[-] Downloaded: refs/heads/master
[+] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[-] Downloaded: logs/refs/heads/master
[+] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[-] Downloaded: objects/01/0dcc30cc1e89344e2bdbd3064f61c772d89a34
[-] Downloaded: objects/00/00000000000000000000000000000000000000
```

> **Extractor**: To extract the downloaded repository and reconstruct the directory structure.

```
cd ../Extractor
./extractor.sh /path/to/download/.git /path/to/extracted/
```

I found interesting files in the /dev/.git/objects/pack

<figure><img src="../../.gitbook/assets/4 (2).png" alt=""><figcaption></figcaption></figure>

lets downloaded all

I found anther tool anther that unpacks all in .git dir lets use it

```
./git_dumper.py  http://10.10.11.177/dev/.git/ ../../updown/
Warning: Destination '../../updown/' is not empty
[-] Testing http://10.10.11.177/dev/.git/HEAD [200]
[-] Testing http://10.10.11.177/dev/.git/ [200]
[-] Fetching .git recursively
[-] Fetching http://10.10.11.177/dev/.git/ [200]
[-] Fetching http://10.10.11.177/dev/.gitignore [404]
[-] http://10.10.11.177/dev/.gitignore responded with status code 404
[-] Fetching http://10.10.11.177/dev/.git/objects/ [200]
[-] Fetching http://10.10.11.177/dev/.git/description [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/ [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/ [200]
[-] Fetching http://10.10.11.177/dev/.git/HEAD [200]
[-] Fetching http://10.10.11.177/dev/.git/info/ [200]
[-] Fetching http://10.10.11.177/dev/.git/index [200]
[-] Fetching http://10.10.11.177/dev/.git/branches/ [200]
[-] Fetching http://10.10.11.177/dev/.git/config [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/ [200]
[-] Fetching http://10.10.11.177/dev/.git/objects/info/ [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/fsmonitor-watchman.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/objects/pack/ [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/HEAD [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/ [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/applypatch-msg.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/info/exclude [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/commit-msg.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-applypatch.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/post-update.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-commit.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-push.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-merge-commit.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-rebase.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/prepare-commit-msg.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/pre-receive.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/push-to-checkout.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/heads/ [200]
[-] Fetching http://10.10.11.177/dev/.git/hooks/update.sample [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/tags/ [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/remotes/ [200]
[-] Fetching http://10.10.11.177/dev/.git/objects/pack/pack-30e4e40cb7b0c696d1ce3a83a6725267d45715da.pack [200]
[-] Fetching http://10.10.11.177/dev/.git/objects/pack/pack-30e4e40cb7b0c696d1ce3a83a6725267d45715da.idx [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/heads/ [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/remotes/ [200]
[-] Fetching http://10.10.11.177/dev/.git/packed-refs [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/heads/main [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/remotes/origin/ [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/heads/main [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/remotes/origin/ [200]
[-] Fetching http://10.10.11.177/dev/.git/refs/remotes/origin/HEAD [200]
[-] Fetching http://10.10.11.177/dev/.git/logs/refs/remotes/origin/HEAD [200]
[-] Sanitizing .git/config
[-] Running git checkout .
Updated 6 paths from the index
```

we found these

<figure><img src="../../.gitbook/assets/5 (2).png" alt=""><figcaption></figcaption></figure>

lets view and see



## admin.php:

```php
<?php
if(DIRECTACCESS){
	die("Access Denied");
}

#ToDo
?>
```

