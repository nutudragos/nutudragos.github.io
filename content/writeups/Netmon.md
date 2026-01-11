---
os: Windows
status: Pwned
tags: [ftp, prtg, cve-2018-9276, password-increment]
aliases:
IP: 10.10.10.152
Secondary IP:
Hostname: NETMON
Domain:
SID:
VM: HackTheBox
Ports: 21, 80, 135, 139, 445, 5985
Notes: Anonymous FTP to C:\, PRTG old backup with password, CVE-2018-9276 RCE
---
# Resolution summary

>[!summary]
>- Anonymous FTP allowed with access to entire C:\ drive
>- Retrieved user.txt flag via FTP
>- Found PRTG Network Monitor running on port 80
>- Discovered old backup file with credentials in ProgramData (hidden folder)
>- Credentials prtgadmin:PrTg@dmin2018 found in configuration backup
>- Incremented year to 2019 based on machine creation date
>- Accessed admin portal with prtgadmin:PrTg@dmin2019
>- Exploited CVE-2018-9276 for SYSTEM shell

## Improved skills

- Anonymous FTP enumeration
- Hidden folder enumeration (ls -la)
- Configuration backup analysis
- Password pattern recognition
- PRTG Network Monitor exploitation
- CVE exploitation

## Used tools

- [[Nmap]]
- FTP client
- CVE-2018-9276 exploit (GitHub)

---

# Information Gathering

Scanned all TCP ports:

```nmap
# Nmap 7.95 scan initiated Thu Sep 4 01:50:20 2025
Nmap scan report for 10.10.10.152
Host is up (0.073s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 02-03-19  12:18AM                 1024 .rnd
| 02-25-19  10:15PM       <DIR>          inetpub
| 07-16-16  09:18AM       <DIR>          PerfLogs
| 02-25-19  10:56PM       <DIR>          Program Files
| 02-03-19  12:28AM       <DIR>          Program Files (x86)
| 02-03-19  08:08AM       <DIR>          Users
|_11-10-23  10:20AM       <DIR>          Windows
80/tcp    open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
|_http-server-header: PRTG/18.1.37.13946
| http-title: Welcome | PRTG Network Monitor (NETMON)
|_Requested resource was /index.htm
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012
```

---

# Enumeration

## Port 21 - FTP

Anonymous FTP access allowed:
```bash
ftp 10.10.10.152
# Username: anonymous
# Password: <blank>
```

**Key Discovery:**
- Full access to C:\ drive
- Can browse entire filesystem
- **Important:** Use `ls -la` to see hidden files/folders

Obtained user.txt flag directly via FTP:
```
C:\Users\Public\user.txt
```

## Port 80 - HTTP (PRTG Network Monitor)

- PRTG Network Monitor 18.1.37.13946
- Paessler bandwidth monitoring software
- Login page accessible
- Default credentials failed

## Configuration File Discovery

Searched for PRTG configuration files via FTP:

1. Initially checked Program Files PRTG folders
2. Realized ProgramData is hidden folder
3. Used `ls -la` to enumerate properly

Found configuration backup:
```
C:\ProgramData\Paessler\PRTG Network Monitor\PRTG Configuration.old.bak
```

Extracted credentials:
```
Username: prtgadmin
Password: PrTg@dmin2018
```

---

# Exploitation

## Credential Analysis

Initial credentials didn't work:
- prtgadmin:PrTg@dmin2018 ❌

Observation:
- Machine created in 2019
- Backup file from 2018
- Password follows year pattern

Tried incremented password:
- prtgadmin:PrTg@dmin2019 ✓

Successfully logged into PRTG admin portal.

## PRTG RCE (CVE-2018-9276)

Exploited authenticated RCE vulnerability:

Exploit used:
https://github.com/A1vinSmith/CVE-2018-9276

1. Configured exploit with credentials
2. Set reverse shell payload
3. Started listener
4. Executed exploit

Obtained reverse shell as **NT AUTHORITY\SYSTEM**

---

# Trophy

>[!todo] **User.txt**
>C:\Users\Public\user.txt (obtained via FTP)

>[!todo] **Root.txt**
>C:\Users\Administrator\Desktop\root.txt

## Lessons Learned

- **Always use `ls -la` with FTP** to see hidden files and folders
- ProgramData is hidden by default on Windows
- Look for backup files with old credentials
- Password patterns often follow dates/years
- Anonymous FTP can provide significant access
