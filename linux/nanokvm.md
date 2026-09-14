# Sipeed NanoKVM

## Download

```shell script
cd /tmp
curl -fsSLO \
    "https://github.com/sipeed/NanoKVM/releases/download/v1.4.3/20260610_NanoKVM_Rev1_4_3.img"
```

## Deploy

```shell script
ls /dev/sd* | sort
# cygpath -w /dev/sdn1

export DEV_LETTER="k"
export IMG_FILE="20260610_NanoKVM_Rev1_4_3.img"

# Then use the `Flash the disk image` code from the `linux_disk_utils.md`
```

## Fix SD card I/O 

```shell script
cat << EOF | tee /etc/fstab
# <file system> <mount pt> <type> <options> <dump> <pass>
/dev/root / ext2 rw,noauto 0 1
proc /proc proc defaults 0 0
devpts /dev/pts devpts defaults,gid=5,mode=620,ptmxmode=0666 0 0
tmpfs /dev/shm tmpfs mode=0777 0 0
tmpfs /run tmpfs mode=0755,nosuid,nodev 0 0
sysfs /sys sysfs defaults 0 0
tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0
tmpfs /var/log tmpfs defaults,noatime 0 0
EOF

shutdown -r now
```