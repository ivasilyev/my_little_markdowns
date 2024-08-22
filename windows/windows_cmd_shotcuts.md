# Reference for Windows GUI tools access using Windows CLI commands

## Absolute path for `cmd`

```shell script
C:\Windows\System32\cmd.exe
```

## Open an elevated Command Prompt

Open CMD window and press `Ctrl + Shift + Enter`

## Process Manager

`taskmgr`

## Performance monitor with tabular view 

`perfmon /res`

## Updater

`wuauclt /updatenow`

## System Configurator

`msconfig`

## File Explorer

`explorer`

## Calculator

`calc`

## Control Panel tools

### Absolute path for `control`

```shell script
C:\Windows\System32\control.exe
```

### Disk Manager

`diskmgmt.msc`

### Service Manager

`services.msc`

### Device Manager

`devmgmt.msc`

### Computer Manager

`compmgmt.msc`

### Local Users and Groups

`lusrmgr.msc`

### Group Policy 

`gpedit.msc`

### Local security settings

`secpol.msc`

### Task Scheduler

`taskschd.msc`

### Shared Folders

`fsmgmt.msc`

## `Rundll32` commands

### Absolute path for `rundll32`

```shell script
C:\Windows\System32\rundll32.exe
```

### Power Options

`rundll32 Shell32.dll,Control_RunDLL powercfg.cpl`

### Program Manager

`runDll32 shell32.dll,Control_RunDLL appwiz.cpl,,0`

### Network Connections

`rundll32 shell32.dll,Control_RunDLL ncpa.cpl`

### Network credentials editor

`rundll32 keymgr.dll, KRShowKeyMgr`

### User environment variables editor

`rundll32 sysdm.cpl,EditEnvironmentVariables`

### User Manager

`rundll32 shell32.dll,Control_RunDLL nusrmgr.cpl`
