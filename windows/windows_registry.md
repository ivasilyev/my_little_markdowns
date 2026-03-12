# Windows registry short reference

## Absolute path for `reg`

```shell script
C:\Windows\System32\reg.exe
```

## Operations (using environment variables as an example)

### Read an entire folder

```shell script
reg query HKEY_CURRENT_USER\Environment
```

### (Over)write the `TestValue` value under the `TestKey` key and with the `REG_EXPAND_SZ` datatype

```shell script
reg add HKEY_CURRENT_USER\Environment /v TestKey /t REG_EXPAND_SZ /d TestValue /f
```

### Delete the `TestKey` key

```shell script
reg delete HKEY_CURRENT_USER\Environment /v TestKey /f
```

## Examples

### Find an old computer hostname

```shell script
reg query HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\SchedulingAgent /v OldName
```

### Enable legacy RDP connections (insecure)

```shell script
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\CredSSP\Parameters /v AllowEncryptionOracle /t REG_DWORD /d 2 
```

### Enable simultaneous WiFi + Ethernet connections

```shell script
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WcmSvc\GroupPolicy /v fMinimizeConnections /t REG_DWORD /d 0 
```

### Hide names and email addresses on logon screen

```shell script
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v dontdisplaylastusername /t REG_DWORD /d 1 /f
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v DontDisplayLockedUserID /t REG_DWORD /d 3 /f
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v dontdisplayusername /t REG_DWORD /d 1 /f
```

### Disable file system path limit

```shell script
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem /v LongPathsEnabled /t REG_DWORD /d 1 /f
```

More workaround:
- Local access: `\\?\e:\Share\`
- Network access: `\\?\UNC\HostName\Share\`
