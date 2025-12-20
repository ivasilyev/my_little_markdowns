# Android setup for a seamless Anydesk experience

## Install apps

- Magisk
- Shizuku
- Appops

## Install Anydesk

- `com.anydesk.anydeskandroid_6.1.12-60112.apk`
- One of the platform-dependent plugins
    - `anydesk-plugins-ad1-latest.apk`
    - `adcontrol 1.1.0`
    - `adconrol-ad1 1.1.2`

## Open App Ops

```text
Shizuku mode
Anydesk plugin
(enable all)

Anydesk

Microphone
Record Audio: Deny

Storage
Read Storage: Deny
Write Storage: Deny

Project Media: Allow
```
## Open Settings

```text
Accessibility
Downloaded services
Anydesk plugin
On (may vary)

Security & Location
Device Admin Apps
Anydesk

Apps & notifications
Special app Access
Battery optimization
All apps
(off) Anydesk
(off) Anydesk plugin

Security
Device Admin Apps
Anydesk
```

## Disable `Start recording or casting` if App Ops does not work

```text
(replace w/ anydesk)
adb shell pm list packages adb shell appops set com.fooview.android.fooview PROJECT_MEDIA allow 
adb shell appops set com.anydesk.anydeskandroid PROJECT_MEDIA allow
```
## Open Anydesk

```text
Security
Interactive Access
Always show icoming session requests

Permissions
Permission Profiles
Permission Profiles
Unattended Access
Profile enabled
Enable unattended access
Set password
Close
```
## Connect from remote device

```text
Start streaming?
Do not ask again
Yes
```
