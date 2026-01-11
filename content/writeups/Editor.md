---
os: Linux
status: Pwned
tags: [xwiki, rce, netdata, cve-2025-24893, cve-2024-32019]
aliases:
IP: 10.10.11.80
Secondary IP:
Hostname: editor.htb
Domain:
SID:
VM: HackTheBox
Ports: 22, 80, 8080
Notes: XWiki unauth RCE, netdata SUID privesc
---
# Resolution summary

>[!summary]
>- XWiki application on port 8080 vulnerable to unauthenticated RCE (CVE-2025-24893)
>- Exploited XWiki for initial access
>- Found plaintext credentials in XWiki config: oliver:theEd1t0rTeam99
>- SSH access as oliver user
>- Enumerated SUID binaries with LinPEAS
>- Found vulnerable ndsudo binary: /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
>- Exploited CVE-2024-32019 for privilege escalation to root

## Improved skills

- XWiki exploitation
- Configuration file analysis
- SUID binary enumeration
- CVE exploitation and compilation
- Netdata privilege escalation

## Used tools

- [[Nmap]]
- XWiki RCE exploit (GitHub)
- LinPEAS
- CVE-2024-32019 POC

---

# Information Gathering

Scanned all TCP ports:

```nmap
Nmap scan report for 10.10.11.80
Host is up (0.075s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
8080/tcp open  http    Jetty 10.0.20
|_http-open-proxy: Proxy might be redirecting requests
| http-webdav-scan:
|   Server Type: Jetty(10.0.20)
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|_  WebDAV type: Unknown
| http-methods:
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
|_http-server-header: Jetty(10.0.20)
| http-robots.txt: 50 disallowed entries
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/
...
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.10.11.80:8080/xwiki/bin/view/Main/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

---

# Enumeration

## Port 80 - HTTP (nginx)

- nginx 1.18.0
- Redirects to editor.htb
- Added to /etc/hosts

## Port 8080 - HTTP (XWiki)

- Jetty 10.0.20
- XWiki application running
- robots.txt with 50 entries
- WebDAV methods enabled

---

# Exploitation

## XWiki Unauthenticated RCE (CVE-2025-24893)

Exploited XWiki vulnerability using:
https://github.com/gunzf0x/CVE-2025-24893

Gained initial shell access.

## Credential Discovery

Enumerated XWiki configuration files and found plaintext credentials:
```
oliver:theEd1t0rTeam99
```

SSH access obtained as oliver user.

---

# Privilege Escalation to root

## Local enumeration

Ran LinPEAS enumeration:
```bash
./linpeas.sh
```

Discovered several SUID binaries including:
```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
```

## Privilege Escalation vector

Exploited netdata ndsudo vulnerability (CVE-2024-32019):

1. Downloaded exploit from:
   https://github.com/AliElKhatteb/CVE-2024-32019-POC

2. Modified IP address in exploit code

3. Compiled exploit:
```bash
gcc exploit.c -o exploit
```

4. Transferred to target and executed

5. Obtained root shell

---

# Trophy

>[!todo] **User.txt**
>/home/oliver/user.txt

>[!todo] **Root.txt**
>/root/root.txt

## Additional Users

```
oliver:x:1000:1000:,,,:/home/oliver:/bin/bash
xwiki:x:997:997:XWiki:/var/lib/xwiki:/usr/sbin/nologin
tomcat:x:998:998:Apache Tomcat:/var/lib/tomcat:/usr/sbin/nologin
netdata:x:996:999:netdata:/opt/netdata:/usr/sbin/nologin
```
