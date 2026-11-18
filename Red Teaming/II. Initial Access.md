# Initial Access

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

### --------------------------------- Password Attack Techniques -----------------------------------

Common attacks: Dictionary, Brute-force, Rule-based, Guessing.

#### **Password Cracking vs Guessing**

| Password Guessing                      | Password Cracking                   |
| -------------------------------------- | ----------------------------------- |
| Online attack                          | Offline/local attack                |
| Tries passwords against login services | Cracks stolen hashes into plaintext |
| Generates login logs                   | No login requests needed            |
| Can cause account lockout              | Usually faster                      |

**Tools:** [Hashcat](https://hashcat.net/hashcat/?utm_source=chatgpt.com) and [John the Ripper](https://www.openwall.com/john/?utm_source=chatgpt.com)

**Remember:** Guessing = Try passwords online. Cracking = Recover passwords from hashes.

### --------------------------------- Password Profiling #1 -----------------------------------

**Default passwords:** Manufacturer-provided passwords (e.g., `admin:admin`, `tomcat:admin`).

Links:\
[Default Passwords 1](https://cirt.net/passwords?utm_source=chatgpt.com) | [2](https://default-password.info/?utm_source=chatgpt.com) | [3](https://datarecovery.com/rd/default-passwords/?utm_source=chatgpt.com)

**Weak / Leaked passwords:** Common or publicly leaked passwords (dumps).

[SkullSecurity](https://www.skullsecurity.org/wiki/Passwords?utm_source=chatgpt.com) | [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords?utm_source=chatgpt.com) | [Leaked Databases](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases?utm_source=chatgpt.com)

#### **Combine Wordlists**

```
cat file1.txt file2.txt file3.txt > combined_list.txt
sort combined_list.txt | uniq -u > cleaned_combined_list.txt
```

#### **Custom Password Wordlist**

**CeWL:** Crawls websites and collects words for targeted wordlists.

```
cewl -w list.txt -d 5 -m 5 http://thm.labs
```

`-w` = Output file, `-d` = Crawl depth, `-m` = Minimum word length.

Target: [https://clinic.thmredteam.com/](https://clinic.thmredteam.com/?utm_source=chatgpt.com) (minimum word length: 8).

#### **Username Wordlist**

Example: `John Smith`

Possible usernames: `john`, `smith`, `jsmith`, `j.smith`, `j-smith`, `j_smith`, `j+smith`, `smithjohn`.

**Tool:** [username_generator](https://github.com/therodri2/username_generator?utm_source=chatgpt.com)

```
git clone https://github.com/therodri2/username_generator.git
cd username_generator
python3 username_generator.py -h
echo "John Smith" > users.lst
python3 username_generator.py -w users.lst
```

`-u` = Also generate uppercase combinations.

### --------------------------------- Password Profiling #2 -----------------------------------

#### **Keyspace Technique – Crunch**

Creates wordlists using specified characters and length.

```
crunch 2 2 01234abcd -o crunch.txt
crunch 8 8 0123456789abcdefABCDEF -o crunch.txt
crunch 6 6 -t pass%%
```

**Pattern symbols:**

| Symbol | Meaning            |
| ------ | ------------------ |
| `@`    | Lowercase letters  |
| `,`    | Uppercase letters  |
| `%`    | Numbers            |
| `^`    | Special characters |

`-t` = Custom pattern, `-o` = Output file.

**Note:** Wordlists can become extremely large.

#### **CUPP**

[GitHub](https://github.com/Mebus/cupp?utm_source=chatgpt.com)

Creates targeted wordlists using personal information (name, birthday, pet, company, etc.).

```
git clone https://github.com/Mebus/cupp.git
cd cupp
python3 cupp.py -h
python3 cupp.py -i
```

**Options:**

| Option | Purpose                                |
| ------ | -------------------------------------- |
| `-i`   | Interactive wordlist generation        |
| `-w`   | Improve existing wordlist              |
| `-l`   | Download wordlists                     |
| `-a`   | Get default credentials from Alecto DB |
| `-v`   | Version                                |
| `-q`   | Quiet mode                             |

**Leet mode:** Replaces letters with numbers (e.g., `a → 4`, `i → 1`).

**Remember:** Crunch = Character combinations. CUPP = Personal-information-based wordlists.


### --------------------------------- Offline Attacks - Dictionary and Brute-Force -----------------------------------

#### **Dictionary Attack**

- Uses a **wordlist** to guess passwords.
- Identify hash type using `hashid` or `hash-identifier`.
- Hashcat example (MD5):

```
hashcat -a 0 -m 0 HASH /usr/share/wordlists/rockyou.txt
```

- `-a 0` = Dictionary attack
- `-m 0` = MD5
- `--show` = Show cracked password

```
hashcat -a 0 -m 0 HASH /usr/share/wordlists/rockyou.txt --show
```

#### **Brute-Force Attack**

- Tries **all possible character combinations**.
- Example: 4-digit PIN = `0000–9999`.
- Hashcat charset:

| Symbol | Characters         |
| ------ | ------------------ |
| `?l`   | Lowercase          |
| `?u`   | Uppercase          |
| `?d`   | Digits             |
| `?h`   | Hex lowercase      |
| `?H`   | Hex uppercase      |
| `?s`   | Special characters |
| `?a`   | All characters     |

Generate 4-digit combinations:

```
hashcat -a 3 ?d?d?d?d --stdout
```

Crack a 4-digit MD5 PIN:

```
hashcat -a 3 -m 0 HASH ?d?d?d?d
```

- `-a 3` = Brute-force attack
- `?d?d?d?d` = Four digits
- `--stdout` = Print combinations to terminal

**Remember:**

- Dictionary = Tries passwords from a wordlist.
- Brute-force = Tries every possible combination.

### --------------------------------- Offline Attacks - Rule-Based -----------------------------------

#### **Rule-Based (Hybrid) Attack**

- Modifies existing wordlist passwords using rules.
- Uses password policy knowledge.
- Example: `password` → `p@ssword`, `Pa$$word`, `Passw0rd`.

**John's rule location:** `/etc/john/john.conf` or `/opt/john/john.conf`

Find available rules:

```
cat /etc/john/john.conf | grep "List.Rules:"
```

Use `best64` rule:

```
john --wordlist=words.txt --rules=best64 --stdout
```

- `--wordlist` = Input wordlist
- `--rules` = Select rules
- `--stdout` = Print generated passwords

Other useful rules: `KoreLogic`, `best64`.

[KoreLogic Rules](https://contest-2010.korelogic.com/rules.html?utm_source=chatgpt.com)

#### **Custom Rules**

Example goal: `[symbol]word[0-9]`

Add to `/etc/john/john.conf`:

```
[List.Rules:THM-Password-Attacks]
Az"[0-9]" ^[!@#$]
```

- `Az` = Use original word.
- `"[0-9]"` = Add one digit at the end.
- `^[!@#$]` = Add a symbol at the beginning.
- `$` instead of `^` = Add symbol at the end.

Use the custom rule:

```
echo "password" > /tmp/single.lst
john --wordlist=/tmp/single.lst --rules=THM-Password-Attacks --stdout
```

**Remember:** Rule-based attack = Existing words + modifications to generate password variations.


### --------------------------------- Online password attacks -----------------------------------

- Guess passwords against **live network services**.
- Common services: FTP, SSH, SMTP, HTTP, VNC, SNMP, POP3.
- Tool: **Hydra**

#### **Hydra Commands**

**FTP**

```
hydra -l ftp -P passlist.txt ftp://10.10.x.x
```

**SMTP**

```
hydra -l email@company.xyz -P wordlist.txt smtp://10.10.x.x -v
```

**SSH**

```
hydra -L users.lst -P wordlist.txt ssh://10.10.x.x -v
```

**HTTP GET Login**

```
hydra -l admin -P wordlist.txt 10.10.x.x http-get-form "/login-get/index.php:username=^USER^&password=^PASS^:S=logout.php" -f
```

Other tools: **Medusa, Ncrack**.


### --------------------------------- Password spray attack -----------------------------------

<img width="1284" height="685" alt="image" src="https://github.com/user-attachments/assets/0e1d104c-b48f-4b77-965c-a34759578f81" />

#### **Password Spraying**

- Uses **one common password against many usernames**.
- Brute-force = Many passwords against one user.
- Spraying = One password against many users.
- Helps avoid account lockout policies.

**Common password patterns:**

- `SeasonYear` → `Spring2021`
- `MonthYear` → `November2020`
- `CompanyNameNumbers` → `TryHackMe01`
- Add symbols if required → `Spring2021!`

**Important:** Enumerate valid usernames before spraying.

#### **SSH – Hydra**

```
hydra -L usernames-list.txt -p Spring2021 ssh://MACHINE_IP
```

- `-L` = Username list
- `-p` = One password

#### **RDP – RDPassSpray**

[GitHub](https://github.com/xFreed0m/RDPassSpray?utm_source=chatgpt.com)

```
python3 RDPassSpray.py -u victim -p 'Spring2021!' -t TARGET:PORT
```

Username list + one password:

```
python3 RDPassSpray.py -U usernames-list.txt -p 'Spring2021!' -d DOMAIN -T RDP_servers.txt
```

#### **Other Services**

- **OWA:** [SprayingToolkit](https://github.com/byt3bl33d3r/SprayingToolkit?utm_source=chatgpt.com), [MailSniper](https://github.com/dafthack/MailSniper?utm_source=chatgpt.com)
- **SMB:** Metasploit `auxiliary/scanner/smb/smb_login`

**Remember:** Password spraying = Many users + One password.





## Phishing

### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
### --------------------------------- Maltego -----------------------------------
