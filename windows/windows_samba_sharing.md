# Tweak Windows SMB clients & servers

Check these services are started:
- DNS Client
- Function Discovery Resource Publication
- SSDP Discovery
- UPnP Device Host

Check the firewalls allow Network Discovery for the active network profile.

Open Network & Sharing Center > Change Advanced Sharing settings):

1. Turn on Network Discovery
2. Turn on file and printer sharing
3. Turn on sharing so anyone with network access can read and write files in the public folders
4. Turn off password protected sharing

# Enable legacy Samba protocols

```powershell
Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName SMB1Protocol
Set-SmbServerConfiguration -EnableSMB2Protocol $true -Confirm:$false
Restart-Computer
```

# Modify File Explorer settings

```shell script
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLinkedConnections /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" /v AllowInsecureGuestAuth /t REG_DWORD /d 1 /f
net stop LanmanWorkstation /y
net start LanmanWorkstation /y

net use * \\host\Share
```

# Clear cached passwords

```shell script
net use * /d
klist purge
reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanServer\Shares /f
```
