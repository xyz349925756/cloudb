```powershell
$manifest = (Get-AppxPackage Microsoft.WindowsStore).InstallLocation + '\AppxManifest.xml' ; Add-AppxPackage -DisableDevelopmentMode -Register $manifest

Get-AppXPackage -AllUsers |Where-Object {$_.InstallLocation -like "*SystemApps*"}
```

https://answers.microsoft.com/zh-hans/windows/forum/all/windows-10/ee8c3cc8-6834-48a0-9bee-46d8ce97a8c3