---
os: Windows
status: Pwned
tags: [windows-xp, ms08-067, ms17-010, smb]
aliases:
IP: 10.10.10.4
Secondary IP:
Hostname: LEGACY
Domain: HTB
SID:
VM: HackTheBox
Ports: 135, 139, 445
Notes: Windows XP SP3, MS08-067 and MS17-010 vulnerable
---
# Resolution summary

>[!summary]
>- Windows XP Professional SP3 identified
>- SMB vulnerable to CVE-2008-4250 (MS08-067) and CVE-2017-0143 (MS17-010)
>- Initially exploited MS17-010 for limited shell
>- Switched to MS08-067 exploit for full administrator access
>- Direct admin rights - both flags obtained immediately

## Improved skills

- Windows XP exploitation
- MS17-010 (EternalBlue) exploitation
- MS08-067 exploitation
- Legacy Windows vulnerability assessment
- NSE script usage for vulnerability detection

## Used tools

- [[Nmap]]
- [[CrackMapExec]]
- MS17-010 exploit (GitHub)
- MS08-067 exploit (GitHub)

---

# Information Gathering

Scanned all TCP ports:

```nmap
nmap 10.10.10.4 -sV -sC -p- -oA nmap_scans/nmap10.10.10.4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-26 03:15 EDT
Nmap scan report for 10.10.10.4
Host is up (0.084s latency).
Not shown: 65532 closed tcp ports (reset)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:94:62:87 (VMware)
| smb-os-discovery:
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2025-07-31T12:17:51+03:00
```

---

# Enumeration

## Port 445 - SMB

Attempted anonymous access:
```bash
netexec smb 10.10.10.4 -u guest -p '' --shares
# Result: STATUS_LOGON_FAILURE
```

### Vulnerability Scanning

```bash
nmap --script smb-vuln* -p 139,445 10.10.10.4
```

**Results:**
- **CVE-2008-4250** (MS08-067) - VULNERABLE
- **CVE-2017-0143** (MS17-010) - VULNERABLE

## System Information

After exploitation, obtained systeminfo:
```
Host Name:                 LEGACY
OS Name:                   Microsoft Windows XP Professional
OS Version:                5.1.2600 Service Pack 3 Build 2600
System type:               X86-based PC
```

---

# Exploitation

## Initial Attempt: MS17-010 (EternalBlue)

Exploited using:
https://github.com/h3x0v3rl0rd/MS17-010

Result:
- Obtained reverse shell
- Limited shell functionality (cd command not working)

## Successful Exploitation: MS08-067

Since OS was confirmed as Windows XP SP3, switched to MS08-067:

Exploit used:
https://gist.github.com/jrmdev/5881544269408edde11335ea2b5438de

1. Generated payload with msfvenom
2. Configured exploit with target details
3. Executed exploit

Result:
- Full administrator privileges granted
- Complete shell functionality
- Both user.txt and root.txt accessible

---

# Trophy

>[!todo] **User.txt**
>C:\Documents and Settings\john\Desktop\user.txt

>[!todo] **Root.txt**
>C:\Documents and Settings\Administrator\Desktop\root.txt

## Notes

- MS17-010 worked but provided limited shell
- MS08-067 provided full admin access on Windows XP
- No privilege escalation needed - direct admin rights
- Windows XP extremely vulnerable to multiple SMB exploits
