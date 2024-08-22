# Windows PowerShell short reference

## Get total number of Cores

```shell script
(Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors
```
