# Tweaks for an `android-x86` VM instance

## Start GUI from command line in Android-x86

Boot into debug mode.

```shell script
mount -o remount,rw /mnt
vi /mnt/grub/menu.lst
```

Remove the `quiet` parameter 
and append into the first option 
(the first line starthing with `kernel`) 
`nomodeset xforcevesa UVESA_MODE=1280x720`

Then restart the VM.

## Enable screen orientation change

```shell script
vi etc/init.sh
```
```
has_sensors = true
```

## Change VM screen resolution

- Calculate the screen options to convert physical pixels to density-independent: `dp = px * (160 / dpi)`
- Open the terminal emulator

```shell script
su
wm size 1280x1024
wm density 240
wm overscan reset
```
