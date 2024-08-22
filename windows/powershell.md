# Windows PowerShell short reference

## Get total number of CPU threads

```shell script
(Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors
```
