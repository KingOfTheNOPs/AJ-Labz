# PowerView

### AD Enumeration With PowerView

Though the below gives a good reperesentation of the commands that usually come in most useful for me, this only scratches the surface of what PowerView can do. PowerView is available [here](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1).

```powershell
# Get all users in the current domain
Get-DomainUser | select -ExpandProperty cn

# Get all computers in the current domain
Get-DomainComputer

# Get all domains in current forest
Get-ForestDomain

# Get domain/forest trusts
Get-DomainTrust
Get-ForestTrust

# Get information for the DA group
Get-DomainGroup "Domain Admins"

# Find members of the DA group
Get-DomainGroupMember "Domain Admins" | select -ExpandProperty membername

# Find interesting shares in the domain, ignore default shares, and check access
Find-DomainShare -ExcludeStandard -ExcludePrint -ExcludeIPC -CheckShareAccess

# Get OUs for current domain
Get-DomainOU -FullData

# Get computers in an OU
# %{} is a looping statement
Get-DomainOU -name Servers | %{ Get-DomainComputer -SearchBase $_.distinguishedname } | select dnshostname

# Get GPOs applied to a specific OU
Get-DomainOU *WS* | select gplink
Get-DomainGPO -Name "{3E04167E-C2B6-4A9A-8FB7-C811158DC97C}"

# Get Restricted Groups set via GPOs, look for interesting group memberships forced via domain
Get-DomainGPOLocalGroup -ResolveMembersToSIDs | select GPODisplayName, GroupName, GroupMemberOf, GroupMembers

# Get the computers where users are part of a local group through a GPO restricted group
Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName

# Find principals that can create new GPOs in the domain
Get-DomainObjectAcl -SearchBase "CN=Policies,CN=System,DC=targetdomain,DC=com" -ResolveGUIDs | ?{ $_.ObjectAceType -eq "Group-Policy-Container" } | select ObjectDN, ActiveDirectoryRights, SecurityIdentifier

# Find principals that can link GPOs to OUs
Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | ? { $_.ObjectAceType -eq "GP-Link" -and $_.ActiveDirectoryRights -match "WriteProperty" } | select ObjectDN, SecurityIdentifier

# Get incoming ACL for a specific object
Get-DomainObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | Select IdentityReference,ActiveDirectoryRights

# Find interesting ACLs for the entire domain, show in a readable (left-to-right) format
Find-InterestingDomainAcl | select identityreferencename,activedirectoryrights,acetype,objectdn | ?{$_.IdentityReferenceName -NotContains "DnsAdmins"} | ft

# Get interesting outgoing ACLs for a specific user or group
# ?{} is a filter statement
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReference -match "Domain Admins"} | select ObjectDN,ActiveDirectoryRights
```

### Lateral Movement Enumeration With PowerView

```powershell
# Find existing local admin access for user (noisy 🚩)
Find-LocalAdminAccess

# Hunt for sessions of interesting users on machines where you have access (also noisy 🚩)
Find-DomainUserLocation -CheckAccess | ?{$_.LocalAdmin -Eq True }

# Look for kerberoastable users
Get-DomainUser -SPN | select name,serviceprincipalname

# Look for AS-REP roastable users
Get-DomainUser -PreauthNotRequired | select name

# Look for interesting ACL within the domain, filtering on a specific user or group you have compromised
## Exploitation depends on the identified ACL, some techniques are discussed in this cheat sheet
## Example for GenericWrite on user: Disable preauth or add SPN for targeted kerberoast (see below)
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "UserOrGroupToQuery"}

# Look for servers with Unconstrained Delegation enabled
## If available and you have admin privs on this server, get user TGT (see below)
Get-DomainComputer -Unconstrained

# Look for users or computers with Constrained Delegation enabled
## If available and you have user/computer hash, access service machine as DA (see below)
Get-DomainUser -TrustedToAuth | select userprincipalname,msds-allowedtodelegateto
Get-DomainComputer -TrustedToAuth | select name,msds-allowedtodelegateto

Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
Get-DomainGroup | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
Get-DomainComputer | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ |Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq$("$env:UserDomain\$env:Username")) {$_}}

```
