---
os: Linux
status: Pwned
tags: [distccd, samba, nmap-privesc]
aliases:
IP: 10.10.10.3
Secondary IP:
Hostname: lame
Domain: hackthebox.gr
SID:
VM: HackTheBox
Ports: 21, 22, 139, 445, 3632
Notes: vsftpd backdoor decoy, distccd RCE, nmap SUID privesc
---
# Resolution summary

>[!summary]
>- Enumerated with [[Nmap]] finding FTP, SSH, SMB, and distccd services
>- vsftpd 2.3.4 on port 21 appeared vulnerable but was a decoy (port 6200 filtered)
>- SMB anonymous login successful but no useful access
>- Exploited distccd v1 on port 3632 (CVE-2004-2687) for initial shell as daemon
>- Enumerated with LinPEAS finding SUID binaries
>- Escalated privileges using nmap interactive mode to root

## Improved skills

- Identifying decoy services
- distccd exploitation
- SUID binary privilege escalation
- GTFOBins usage for privilege escalation

## Used tools

- [[Nmap]]
- LinPEAS
- Python POC for distccd RCE

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap 10.10.10.3 -sV -sC -p- -oA nmap_scans/10.10.10.3-sV-sC-p-
```

Enumerated open TCP ports:

```nmap
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-27 05:31 EDT
Nmap scan report for 10.10.10.3
Host is up (0.054s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.16.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-07-27T05:33:45-04:00
|_clock-skew: mean: 2h00m21s, deviation: 2h49m46s, median: 18s
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

---

# Enumeration

## Port 21 - FTP (vsftpd 2.3.4)

- vsftpd 2.3.4 identified - vulnerable to CVE-2011-2512 (backdoor)
- Attempted multiple POCs but port 6200 is filtered (required for exploit)
- Concluded vsftpd is likely a decoy

## Port 139/445 - SMB (Samba 3.0.20-Debian)

- Anonymous login successful
- Several files and folders discovered but no access granted
- SMB also appears to be a decoy

## Port 3632 - distccd

- distccd v1 running
- Vulnerable to CVE-2004-2687 (Remote Code Execution)
- Successfully exploited for initial access

---

# Exploitation

## distccd Remote Code Execution (CVE-2004-2687)

Used Python POC from:
https://gist.githubusercontent.com/adityatelange/3c067c7a126b93d2eaba195b65308577/raw/38637521969efb5797fb8020ab8085c0dd3f22b4/distccd_rce_CVE-2004-2687.py

Successfully executed commands and obtained reverse shell as `daemon` user.

---

# Privilege Escalation to root

## Local enumeration

Ran LinPEAS enumeration script which identified:
- Multiple SUID binaries
- nmap with SUID bit set
- Older version of nmap with interactive mode

## Privilege Escalation vector

Exploited nmap interactive mode to spawn root shell:

```bash
nmap --interactive
!sh
```

Reference: https://gtfobins.github.io/gtfobins/nmap/

---

# Trophy

>[!todo] **User.txt**
>Obtained via daemon shell

>[!todo] **Root.txt**
>Obtained via nmap privilege escalation

**/etc/shadow**

```bash
root:$1$...
```
