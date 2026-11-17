# Red Team Fundamentals

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

