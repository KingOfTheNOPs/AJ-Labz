---
description: Just some rick references
---

# AD Enumeration



## ActiveDirectory Enumeration Notes

Get information about an AD group

```text
Get-ADGroup -Identity "<GROUP NAME" -Properties *
```

View a user's current rights

```text
whoami /priv
```

Check if RSAT installed

```text
Get-WindowsCapability -Name RSAT* -Online | Select-Object -Property Name, State
```

Install all RSAT tools

```text
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability –Online
```

LDAP query to return all AD groups

```text
Get-ADObject -LDAPFilter '(objectClass=group)' | select cn
```

Find another disabled user \(first.last\).

```text
Get-ADUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=2)' | select name

#OID REFERENCE https://ldapwiki.com/wiki/OID
```

Run a utility as another user

```text
runas /netonly /user:DOMAIN\USERNAME powershell
```

Count all users in an OU

```text
 Get-ADOrganizationalUnit -Filter 'Name -like "*"' | Format-Table Name, DistinguishedName -A

(Get-ADUser -SearchBase "OU=<OU>,DC=<DOMAIN>,DC=<DOMAIN>" -Filter *).count
```

Count all users in Domain

```text
(Get-ADUser -Filter *).count
```

Count all computers in Domain

```text
(Get-ADComputer -Filter *).count
```

Count all groups in Domain

```text
(Get-ADGroup -Filter *).count
```

Query for installed software

```text
get-ciminstance win32_product | fl
```

Get hostnames with the word "SQL" in their hostname; Change SQL for whatever hostname you're looking for

```text
Get-ADComputer -Filter "DNSHostName -like 'SQL*'"
```

Get all administrative groups

```text
Get-ADGroup -Filter "adminCount -eq 1" | select Name
```

Find admin users that don't require Kerberos Pre-Auth

```text
Get-ADUser -Filter {adminCount -eq '1' -and DoesNotRequirePreAuth -eq 'True'}
```

Get SID for computer

```text
Get-ADComputer -Filter {Name -like "<HOSTNAME>"} | Select Name,SID | fl
```

Enumerate UAC values for admin users

```text
Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol
```

Get AD groups using WMI

```text
Get-WmiObject -Class win32_group -Filter "Domain='INLANEFREIGHT'"
```

Use ADSI to search for all computers

```text
([adsisearcher]"(&(objectClass=Computer))").FindAll()
```

Find all trusted users or computers marked trusted for delegation

```text
Get-ADUser -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select Name,memberof, servicePrincipalName,TrustedForDelegation | fl

Get-ADComputer -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select DistinguishedName,servicePrincipalName,TrustedForDelegation | fl
```

Find users with blank passwords

```text
Get-AdUser -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))(adminCount=1)' -Properties * | select name,memberof | fl
```

Recursive search of groups a user is apart of

```text
Get-ADGroup -Filter 'member -RecursiveMatch "CN=<USER>,OU=<ORGANIZATIONAL UNIT>,DC=<DOMAIN>,DC=<DOMAIN>"' | select name

Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=<USER>,OU=<ORGANIZATIONAL UNIT>,DC=<DOMAIN>,DC=<DOMAIN>)' |select Name
```

UserAccountControl Attributes

![UAC Attributes](../.gitbook/assets/image%20%28168%29.png)

```text
Get-ADUser -Filter {adminCount -gt 0} -Properties admincount,useraccountcontrol | select Name,useraccountcontrol
```



