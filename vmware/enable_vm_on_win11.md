# Enable virtualization for 3rd-party VMs on Windows 11

Based on [the forum post](https://community.broadcom.com/vmware-cloud-foundation/discussion/windows-11-24h2-hsot-how-to-disable-virtual-based-security).

## Manage Group policies editor 

```text
cmd
gpedit
Computer Configuration
Admininistrative Templates
System
Device Guard
Turn on Virtualization Base Security: Disable
```

## Turn off all options in Core isolation

```text
cmd
start windowsdefender:
Device security
Core isolation
Disable all options
```

## Manage Windows features

```text
cmd
optionalfeatures
Disable Hyper-V, Virtual machine plafrorm, Windows subsystem for Linux
```

## Run cmd by an administrator

```shell script
echo Disable Credential Guard
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa" /v LsaCfgFlags /t REG_DWORD /d 0 /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard" /v LsaCfgFlags /t REG_DWORD /d 0 /f

echo Disable Credential Guard with UEFI lock
mountvol X: /s
copy %WINDIR%\System32\SecConfig.efi X:\EFI\Microsoft\Boot\SecConfig.efi /Y
bcdedit /create {0cb3b571-2f2e-4343-a879-d86a476d7215} /d "DebugTool" /application osloader
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} path "\EFI\Microsoft\Boot\SecConfig.efi"
bcdedit /set {bootmgr} bootsequence {0cb3b571-2f2e-4343-a879-d86a476d7215}
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} loadoptions DISABLE-LSA-ISO
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} device partition=X:
mountvol X: /d

echo Disable VBS with Registry settings
reg delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v EnableVirtualizationBasedSecurity /f
reg delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v RequirePlatformSecurityFeatures /f
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} loadoptions DISABLE-LSA-ISO,DISABLE-VBS
bcdedit /set vsmlaunchtype off
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v Enabled /t REG_DWORD /d 0 /f
```

## Restart PC

- Restart the device
- Before the OS boots, a prompt **must** appear, notifying that UEFI was modified, and asking for confirmation.
- Press the `F3`  or `Win` keys and press `Enter` to continue.
