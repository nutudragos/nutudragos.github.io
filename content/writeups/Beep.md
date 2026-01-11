---
os: Linux
status: Pwned
tags: [elastix, lfi, searchsploit, nmap-privesc]
aliases:
IP: 10.10.10.7
Secondary IP:
Hostname: beep.localdomain
Domain:
SID:
VM: HackTheBox
Ports: 22, 25, 80, 110, 111, 143, 443, 793, 993, 995, 3306, 4190, 4445, 4559, 5038, 10000
Notes: Elastix login, LFI to config, nmap SUID privesc
---
# Resolution summary

>[!summary]
>- Extensive enumeration revealed Elastix web application on ports 80/443
>- Used SearchSploit 37637 LFI vulnerability to read config files
>- Extracted admin password from configuration
>- Utilized SearchSploit 37637 to obtain user shell
>- Escalated to root using nmap interactive shell

## Improved skills

- Elastix enumeration and exploitation
- Local File Inclusion (LFI) attacks
- SearchSploit usage
- Configuration file analysis
- Nmap interactive mode privilege escalation

## Used tools

- [[Nmap]]
- SearchSploit
- Burp Suite

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap 10.10.10.7 -sV -sC -p- -T4 -oA nmap_scans/10.10.10.7-sV-sC-p-
```

Enumerated open TCP ports:

```nmap
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-11 01:58 EDT
Nmap scan report for 10.10.10.7
Host is up (0.089s latency).
Not shown: 65519 closed tcp ports (reset)
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http       Apache httpd 2.2.3
|_http-title: Did not follow redirect to https://10.10.10.7/
|_http-server-header: Apache/2.2.3 (CentOS)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
111/tcp   open  rpcbind    2 (RPC #100000)
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| http-robots.txt: 1 disallowed entry
|_/
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
793/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix
```

---

# Enumeration

## Port 80/443 - HTTP/HTTPS (Elastix)

- Apache httpd 2.2.3 running on CentOS
- Elastix login page on HTTPS
- robots.txt present with disallowed entry
- Attempted various login methods without success

Searched for Elastix vulnerabilities:
```bash
searchsploit elastix
```

Found multiple vulnerabilities including LFI (37637).

---

# Exploitation

## Elastix LFI (SearchSploit 37637)

Used Local File Inclusion vulnerability to read configuration files:

1. Leveraged LFI in Burp Suite to read Elastix config
2. Extracted admin password from configuration file
3. Attempted access through web interface - spent time exploring

Eventually used the same SearchSploit 37637 exploit to:
1. Obtain initial user shell
2. Follow POC instructions for privilege escalation

---

# Privilege Escalation to root

## Local enumeration

System running older software versions with SUID binaries.

## Privilege Escalation vector

Used nmap interactive shell as described in SearchSploit 37637:

```bash
nmap --interactive
!sh
```

Obtained root shell through nmap SUID exploitation.

---

# Trophy

>[!todo] **User.txt**
>Obtained via LFI exploitation

>[!todo] **Root.txt**
>Obtained via nmap privilege escalation
