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
