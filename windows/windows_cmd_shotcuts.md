# Reference for Windows GUI tools access using Windows CLI commands

## Absolute path for `cmd`

```shell script
C:\Windows\System32\cmd.exe
```

## Open an elevated Command Prompt

Open CMD window and press `Ctrl + Shift + Enter`

## Device Manager

`devmgmt.msc`

## Computer Manager

`compmgmt.msc`

## Process Manager

`taskmgr`

## Performance monitor with tabbed view 

`perfmon /res`

## Task Scheduler

`taskschd.msc`

## Windows updater

`wuauclt.exe /updatenow`

## Network credentials editor

`rundll32.exe keymgr.dll, KRShowKeyMgr`

## User environment variables editor

`rundll32 sysdm.cpl,EditEnvironmentVariables`
