# Windows PowerShell short reference

## Absolute path for PowerShell

```text
>where powershell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

## Execute a PowerShell command from `cmd.exe`

```shell script
powershell -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -Command "echo \"Hello World!\""
```

## Get total number of CPU threads

```shell script
(Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors
```

## Create shortcut

```shell script
New-Item -ItemType SymbolicLink -Path "%USERPROFILE%\Desktop" -Name "Calculator.lnk" -Value "C:\Windows\System32\calc.exe"
```

## Minimize all windows and show the Desktop

```shell script
(New-Object -ComObject shell.application).toggleDesktop()
```

## Add Windows Defender exclusion

```shell script
Add-MpPreference -ExclusionPath "C:\program_dir\program.exe"
Add-MpPreference -ExclusionPath "C:\program_dir"
```

## Import task XML into the Windows Scheduler

```shell script
$TaskName = '<task_name>'; $TaskXml = [xml](Get-Content '<task_file.xml>'); Register-ScheduledTask -TaskName $TaskName -Xml $TaskXml.OuterXml; Get-ScheduledTask -TaskName $TaskName
```
