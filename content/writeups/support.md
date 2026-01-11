---
os: Windows
status: Pwned
tags: [active-directory, smb, ldap, dnspy, reversing]
aliases:
IP: 10.10.11.174
Secondary IP:
Hostname: support.htb
Domain: support.htb
SID:
VM: HackTheBox
Ports: 445, 389, 3268, 5985
Notes: SMB anonymous, .NET binary reversing, LDAP password extraction
---
# Resolution summary

>[!summary]
>- Mounted SMB share support-tools with anonymous access
>- Downloaded .NET binaries for analysis
>- Reversed .NET binary using DNSpy/ILSpy to find hardcoded LDAP password
>- Used credentials to enumerate LDAP
>- Extracted user password from LDAP info field
>- Obtained WinRM access with discovered credentials
>- Enumerated Active Directory for privilege escalation

## Improved skills

- SMB share mounting in Linux
- .NET binary reverse engineering
- DNSpy and ILSpy usage
- LDAP enumeration with ldapsearch
- Active Directory enumeration
- Credential extraction from compiled binaries

## Used tools

- [[SMBClient]]
- [[CrackMapExec]]
- mount (CIFS)
- DNSpy / ILSpy
- ldapsearch
- evil-winrm

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- 10.10.11.174 -oA nmap_scans/support
```

Enumerated open TCP ports:

```bash
# Active Directory environment
# Port 445 - SMB
# Port 389 - LDAP
# Port 3268 - Global Catalog
# Port 5985 - WinRM
```

---

# Enumeration

## Port 445 - SMB

### Default Domain Controller Shares

Standard DC shares:
- C$
- ADMIN$
- IPC$
- SYSVOL
- NETLOGON

### Anonymous Enumeration

```bash
crackmapexec smb 10.10.11.174 -u "anonymous" -p "" --shares
```

Found accessible share:
- support-tools (READ access)

### Mounting SMB Share

**Important:** In Linux, mount shares only under /mnt

```bash
sudo mount -t cifs -o username="john" //10.10.11.174/support-tools /mnt/support-tools-share
```

**Note:** Username parameter required even for anonymous access.

Downloaded .NET binaries for analysis.

## Binary Analysis

### Tools for .NET Reverse Engineering

- **DNSpy** - Returns source code for .NET binaries
- **ILSpy** - Alternative .NET decompiler

Analyzed downloaded binaries and found hardcoded LDAP credentials:
```
ldap@support.htb:nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

## Port 389 - LDAP

### LDAP Enumeration

Used discovered credentials to enumerate LDAP:

```bash
ldapsearch -x -H ldap://support.htb -D "ldap@support.htb" -w "nvEfEK16^1aM4\$e7AclUf8x\$tRWxPWO1%lmz" -b "dc=support,dc=htb"
```

Enumerated specific user:
```bash
ldapsearch -x -H ldap://support.htb -D "ldap@support.htb" -w "nvEfEK16^1aM4\$e7AclUf8x\$tRWxPWO1%lmz" -b "dc=support,dc=htb" "(sAMAccountName=support)"
```

Found password in user info/description field.

---

# Exploitation

## WinRM Access

Connected with discovered credentials:
```bash
evil-winrm -i support.htb -u support -p <password>
```

Obtained user shell.

---

# Privilege Escalation

## Local enumeration

Enumerated Active Directory:
- Groups and memberships
- User permissions
- Service accounts
- Exploitable misconfigurations

*Further privilege escalation details would be documented here based on enumeration findings*

---

# Trophy

>[!todo] **User.txt**
>C:\Users\support\Desktop\user.txt

>[!todo] **Root.txt**
>C:\Users\Administrator\Desktop\root.txt

## Key Notes

- SMB share mounting requires username parameter
- .NET binaries can contain hardcoded secrets
- LDAP user info fields may contain sensitive data
- DNSpy/ILSpy essential for .NET reverse engineering
