# Spectra
## Nmap Scan

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
let's open the following link in the browser : http://10.10.10.229

![alt text](https://pencer.io/assets/images/2021-03-21-16-46-59.png)

a simple page with two links, Software Issue Tracking and test.
The test link redirect us to http://10.10.10.229/testing/index.php, in this page we see an error database connection.
When we open http://10.10.10.229/testing, we see a wordpress projet, it's like a testing wordpress projet.
There is a file called wp-config.php.save, it's a configuration file in wordpress.
There is some credentials in this file, like ```define( 'DB_PASSWORD', '<<HIDDEN>>' );```



mysql --host=spectra.htb --user=devtest --password=devteam01 dev
SummerHereWeCome!!

