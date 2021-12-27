# Enumeration Commands

## ActiveDirectory Enumeration Notes

Get information about an AD group

```
Get-ADGroup -Identity "<GROUP NAME" -Properties *
```

View a user's current rights

```
whoami /priv
```

Check if RSAT installed

```
Get-WindowsCapability -Name RSAT* -Online | Select-Object -Property Name, State
```

Install all RSAT tools

```
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
```

LDAP query to return all AD groups

```
Get-ADObject -LDAPFilter '(objectClass=group)' | select cn
```

Find another disabled user (first.last).

```
Get-ADUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=2)' | select name

#OID REFERENCE https://ldapwiki.com/wiki/OID
```

Run a utility as another user

```
runas /netonly /user:DOMAIN\USERNAME powershell
```

Count all users in an OU

```
 Get-ADOrganizationalUnit -Filter 'Name -like "*"' | Format-Table Name, DistinguishedName -A

(Get-ADUser -SearchBase "OU=<OU>,DC=<DOMAIN>,DC=<DOMAIN>" -Filter *).count
```

Count all users in Domain

```
(Get-ADUser -Filter *).count
```

Count all computers in Domain

```
(Get-ADComputer -Filter *).count
```

Count all groups in Domain

```
(Get-ADGroup -Filter *).count
```

Query for installed software

```
get-ciminstance win32_product | fl
```

Get hostnames with the word "SQL" in their hostname; Change SQL for whatever hostname you're looking for

```
Get-ADComputer -Filter "DNSHostName -like 'SQL*'"
```

Get all administrative groups

```
Get-ADGroup -Filter "adminCount -eq 1" | select Name
```

Find admin users that don't require Kerberos Pre-Auth

```
Get-ADUser -Filter {adminCount -eq '1' -and DoesNotRequirePreAuth -eq 'True'}
```

Get SID for computer

```
Get-ADComputer -Filter {Name -like "<HOSTNAME>"} | Select Name,SID | fl
```

Enumerate UAC values for admin users

```
Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol
```

Get AD groups using WMI

```
Get-WmiObject -Class win32_group -Filter "Domain='INLANEFREIGHT'"
```

Use ADSI to search for all computers

```
([adsisearcher]"(&(objectClass=Computer))").FindAll()
```

Find all trusted users or computers marked trusted for delegation

```
Get-ADUser -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select Name,memberof, servicePrincipalName,TrustedForDelegation | fl

Get-ADComputer -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select DistinguishedName,servicePrincipalName,TrustedForDelegation | fl
```

Find users with blank passwords

```
Get-AdUser -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))(adminCount=1)' -Properties * | select name,memberof | fl
```

Recursive search of groups a user is apart of

```
Get-ADGroup -Filter 'member -RecursiveMatch "CN=<USER>,OU=<ORGANIZATIONAL UNIT>,DC=<DOMAIN>,DC=<DOMAIN>"' | select name

Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=<USER>,OU=<ORGANIZATIONAL UNIT>,DC=<DOMAIN>,DC=<DOMAIN>)' |select Name
```

UserAccountControl Attributes

![UAC Attributes](<../../.gitbook/assets/image (168).png>)

```
Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol | select Name,useraccountcontrol
```

Password Not Required

```
Get-ADUser -Filter {PasswordNotRequired -eq $true}
```

Nested Groups

```
Get-ADGroup -filter * -Properties MemberOf | Where-Object {$_.MemberOf -ne $null} | Select-Object Name,MemberOf
```
