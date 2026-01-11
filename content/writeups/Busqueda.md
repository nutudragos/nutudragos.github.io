---
os: Linux
status: Pwned
tags: [searchor, cmd-injection, gitea, docker, sudo-abuse]
aliases:
IP:
Secondary IP:
Hostname: searcher.htb
Domain:
SID:
VM: HackTheBox
Ports: 80, 222, 3000, 5000
Notes: Searchor 2.4.0 RCE, Gitea config extraction, relative path sudo abuse
---
# Resolution summary

>[!summary]
>- Exploited Searchor 2.4.0 arbitrary command injection on port 80
>- Enumerated listening ports finding Gitea on port 3000
>- Found plaintext credentials in .git/config: cody:jh1usoih2bkjaspwe92
>- Used credentials to SSH as svc user
>- Discovered sudo privilege to run system-checkup.py as root
>- Extracted database password via docker-inspect command
>- Accessed Gitea as administrator to review source code
>- Exploited relative path vulnerability in system-checkup.py
>- Created malicious script in /dev/shm to gain root shell

## Improved skills

- Searchor exploitation
- Internal port enumeration
- Git configuration analysis
- Docker inspection
- Sudo privilege escalation
- Relative path exploitation
- GitLab/Gitea enumeration

## Used tools

- [[Nmap]]
- Searchor exploit (GitHub)
- ss (socket statistics)
- docker-inspect
- jq

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- <IP> -oA nmap_scans/busqueda
```

Enumerated open TCP ports:

```bash
# Port 80 - HTTP (Searchor 2.4.0)
```

---

# Enumeration

## Port 80 - HTTP (Searchor)

- Searchor version 2.4.0 identified
- Added searcher.htb to /etc/hosts
- Searchor <= 2.4.2 vulnerable to arbitrary command injection

---

# Exploitation

## Searchor 2.4.0 RCE

Exploited vulnerability using:
https://github.com/nikn0laty/Exploit-for-Searchor-2.4.0-Arbitrary-CMD-Injection

Obtained initial shell access.

---

# Privilege Escalation to root

## Local enumeration

Enumerated listening ports:
```bash
ss -lntp
```

Found services:
- Port 5000 - Searchor
- Port 3000 - Gitea
- Port 222 - SSH

Checked Apache configuration:
```bash
cat /etc/apache2/sites-available/*
```

Discovered gitea.searcher.htb (added to /etc/hosts).

Found .git directory in /var/www/app:
```bash
cat /var/www/app/.git/config
```

Extracted credentials:
- cody:jh1usoih2bkjaspwe92
- Works for SSH as svc user: svc:jh1usoih2bkjaspwe92

## Privilege Escalation vector

Checked sudo permissions:
```bash
sudo -l
# Can run: /usr/bin/python3 /opt/scripts/system-checkup.py * as root
```

Explored system-checkup.py usage:
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py help
```

Used docker-inspect to extract database credentials:
```bash
sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .Config}}' 960
```

Parsed output with jq:
```bash
echo -n '<output>' | jq .
```

Found Gitea database password:
- GITEA__database__PASSWD=yuiu1hoiu4i5ho1uh

Logged into Gitea as administrator and reviewed system-checkup.py source code.

Discovered vulnerability: script runs from relative path instead of absolute path.

Created malicious script in /dev/shm:
```bash
cd /dev/shm
# Created script with same name
# Ran system-checkup.py with appropriate argument
```

Obtained root reverse shell.

---

# Trophy

>[!todo] **User.txt**
>Obtained via SSH as svc

>[!todo] **Root.txt**
>Obtained via relative path exploitation
