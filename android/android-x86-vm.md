# Tweaks for an `android-x86` VM instance

## Start GUI from command line in Android-x86

```shell script
mount -o remount,rw /mnt
vi /mnt/grub/menu.lst
```

Remove the `quiet` parameter 
and append into the first option 
(the first line starthing with `kernel`) 
`nomodeset xforcevesa UVESA_MODE=1280x720

```
reboot -f
```
