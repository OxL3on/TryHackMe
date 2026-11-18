<img width="920" height="560" alt="image" src="https://github.com/user-attachments/assets/fc291bef-fc05-4d4d-8647-56e4930f23de" /># Red Team Fundamentals

## Red Team Engagements

#### ----------------------------------- Defining Scope and Objectives -----------------------------------

**Objectives:** What the client wants to achieve (find weaknesses, test EDR, SIEM, detection, response, and data exposure).

**Penetration Test:** General security testing using standard TTPs (TTPs describe how an attacker performs an attack. TTP = Tactics, Techniques, and Procedures.) .

**Adversary Emulation:** Imitating the real-world attack methods of a specific threat actor (hacker group) to test an organization's security.

**Scope:** Defines what you can and cannot attack. Only the client sets it.

**Scope examples:** Allowed/forbidden IPs, no downtime, no DDoS, no harmful malware, no PII exfiltration.

**White Card:** Permission to use information or assistance from the client during an exercise.

**Engagement Plan:** Detailed plan explaining how to achieve objectives while following the scope.

**Remember:** Objectives = WHAT to achieve. Scope = WHAT is allowed. Plan = HOW to achieve it.


#### ----------------------------------- Rules of Engagement -----------------------------------

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


#### ----------------------------------- Campaign Planning -----------------------------------

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


#### ----------------------------------- Engagement Documentation -----------------------------------

**Engagement Documentation:** Officially records campaign planning. It can be a formal document or even a simple email.

### **Engagement Plan**

| Component     | Purpose                                                     |
| ------------- | ----------------------------------------------------------- |
| CONOPS        | Non-technical overview of how Red Team achieves objectives. |
| Resource Plan | Timeline, personnel, hardware, cloud requirements.          |

### **Operations Plan**

| Component              | Purpose                              |
| ---------------------- | ------------------------------------ |
| Personnel              | Required employees.                  |
| Stopping Conditions    | When and why to stop the engagement. |
| RoE (Optional)         | Rules of Engagement.                 |
| Technical Requirements | Knowledge needed by Red Team.        |

### **Mission Plan**

| Component                    | Purpose                                   |
| ---------------------------- | ----------------------------------------- |
| Command Playbooks (Optional) | Exact commands, tools, when, why and how. |
| Execution Times              | When to start each stage.                 |
| Responsibilities/Roles       | Who does what and when.                   |

### **Remediation Plan (Optional)**

| Component                | Purpose                                 |
| ------------------------ | --------------------------------------- |
| Report                   | Engagement summary and findings.        |
| Remediation/Consultation | How the client fixes discovered issues. |

**Remember:** Engagement = Overview. Operations = Preparation. Mission = Execution. Remediation = Reporting and fixing issues.


#### ----------------------------------- Concept of Operations -----------------------------------

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


#### ----------------------------------- Resource Plan -----------------------------------

**Resource Plan:** Extends CONOPS with specific dates, required knowledge, and resources needed for the engagement.

**Main Components:**

| Component                     | Purpose                                                         |
| ----------------------------- | --------------------------------------------------------------- |
| Header                        | Writer, dates, customer                                         |
| Engagement Dates              | Recon, initial compromise, post-exploitation, persistence dates |
| Knowledge Required (Optional) | Skills needed for each attack phase                             |
| Resource Requirements         | Personnel, hardware, cloud, and other resources                 |

**Remember:** CONOPS = Overall overview. Resource Plan = What resources, skills, and dates are needed.


#### ----------------------------------- Operations Plan -----------------------------------

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


#### ----------------------------------- Mission Plan -----------------------------------

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


#### ----------------------------------- Mission Plan -----------------------------------

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

#### ----------------------------------- What is Threat Intelligence -----------------------------------

**Threat Intelligence (CTI):** Collecting and analyzing information about attackers to understand their activities and techniques.

**IOC (Indicators of Compromise):** Traces left by attackers, such as IPs, domains, files, and strings.

**ISAC:** Information Sharing and Analysis Center – shares threat intelligence like IOCs and TTPs.


#### ----------------------------------- Applying Threat Intel to the Red Team -----------------------------------

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


#### ----------------------------------- The TIBER-EU Framework -----------------------------------

**TIBER-EU:** Threat Intelligence-Based Ethical Red Teaming. A framework developed by the European Central Bank (ECB).

**Purpose:** Helps financial institutions test and improve their security against sophisticated cyberattacks using threat intelligence.

**Key Difference:** Its Testing phase uses threat intelligence to guide Red Team attacks.

**Remember:** TIBER-EU = A best-practice framework that uses real-world threat intelligence to make Red Team testing realistic.

**References:**

[ECB TIBER-EU White Paper (PDF)](https://www.ecb.europa.eu/pub/pdf/other/ecb.tiber_eu_framework.en.pdf?utm_source=chatgpt.com)

[CREST TIBER-EU](https://www.crest-approved.org/membership/tiber-eu/?utm_source=chatgpt.com)


#### ----------------------------------- TTP Mapping -----------------------------------

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


#### ----------------------------------- Other Red Team Applications of CTI -----------------------------------

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

#### ----------------------------------- Introduction -----------------------------------

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


#### ----------------------------------- Critical Information Identification -----------------------------------

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


#### ----------------------------------- Threat Analysis -----------------------------------

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


#### ----------------------------------- Vulnerability Analysis -----------------------------------

**OPSEC Vulnerability:** A weakness that allows an adversary to discover critical information, analyse it, and disrupt your operation.

**Examples:**

1\. **Same IP Address:** Using one IP for Nmap, phishing, and Metasploit. If Blue Team blocks that IP, all activities fail.

2\. **Unsecured Database:** Attackers may steal phishing victim credentials and compromise the client's network.

3\. **Social Media Leak:** Revealing the client's name can help Blue Team prepare for the Red Team's activities.

**Remember:** OPSEC Vulnerability = A weakness that exposes your critical information and allows adversaries to disrupt your operation.


#### ----------------------------------- Risk Assessment -----------------------------------

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


#### ----------------------------------- Countermeasures -----------------------------------

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







