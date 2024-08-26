# Windows PowerShell short reference

## Get total number of CPU threads

```shell script
(Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors
```

## Create shortcut

```shell script
New-Item -ItemType SymbolicLink -Path "%USERPROFILE%\Desktop" -Name "Calculator.lnk" -Value "C:\Windows\System32\calc.exe"
```
