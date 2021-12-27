# Disable AV

### Backstab

Backstab is a tool capable of killing antimalware protected processes by leveraging sysinternals’ Process Explorer (ProcExp) driver, which is signed by Microsoft.

{% embed url="https://github.com/Yaxser/Backstab" %}

### Disable Defender

Assuming tamper protection is off

```
Set-MpPreference -DisableIntrusionPreventionSystem $true -DisableIOAVProtection $true -DisableRealtimeMonitoring $true
```
