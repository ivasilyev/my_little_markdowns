# Linux disk editing utils

## Re-create & mount a disk partittion as NTFS

```shell script
sudo apt-get install \
    --yes \
    ntfs-3g    

export DEVICE_LETTER="b"
export DEVICE_LABEL="mylabel_b"

export DEVICE_NAME="/dev/sd${DEVICE_LETTER}"
export DEVICE_PARTITION="${DEVICE_NAME}1"
export MOUNT_DIR="/data/${DEVICE_LETTER}"

echo View disk drives
sudo lsblk -p

echo Process the disk "${DEVICE_NAME}"
sudo umount -l "${DEVICE_PARTITION}"

echo Delete all partitions
sudo wipefs -af "${DEVICE_NAME}"

echo Create a new MBR disk label
sudo parted "${DEVICE_NAME}" mklabel msdos

echo Create a new NTFS disk partition
sudo parted -a opt "${DEVICE_NAME}" mkpart primary ntfs 0% 100%

echo yes | sudo mkntfs -c 512 -L "${DEVICE_LABEL}" "${DEVICE_PARTITION}" 

echo Create mount point
sudo mkdir -p "${MOUNT_DIR}"
sudo chmod 777 "${MOUNT_DIR}"

echo Mount drive
sudo mount "${DEVICE_PARTITION}" "${MOUNT_DIR}"
touch "${MOUNT_DIR}/test"
rm -f "${MOUNT_DIR}/test"
ls "${MOUNT_DIR}"

echo Add persistent mount
printf "\nUUID=$(sudo blkid -s UUID -o value "${DEVICE_PARTITION}") ${MOUNT_DIR} ext4 defaults 0 0\n" \
| sudo tee -a "/etc/fstab"
## sudo nano "/etc/fstab"
```

## Re-create & mount a disk partittion as EXT4

```shell script
export DEVICE_LETTER="a"
export DEVICE_LABEL="mylabel_a"

export DEVICE_NAME="/dev/sd${DEVICE_LETTER}"
export DEVICE_PARTITION="${DEVICE_NAME}1"
export MOUNT_DIR="/data/${DEVICE_LETTER}"

echo View disk drives
sudo lsblk -p

echo Process the disk "${DEVICE_NAME}"
sudo umount -l "${DEVICE_PARTITION}"

echo Delete all partitions
sudo wipefs -af "${DEVICE_NAME}"

echo Create a new MBR disk label
sudo parted "${DEVICE_NAME}" mklabel msdos

echo Create a new EXT4 disk partition
sudo parted -a opt "${DEVICE_NAME}" mkpart primary ext4 0% 100%
echo yes | sudo mkfs.ext4 -v -b 1024 -L "${DEVICE_LABEL}" "${DEVICE_PARTITION}" 

echo Create mount point
sudo mkdir -p "${MOUNT_DIR}"
sudo chmod 777 "${MOUNT_DIR}"

echo Mount drive
sudo mount "${DEVICE_PARTITION}" "${MOUNT_DIR}"
touch "${MOUNT_DIR}/test"
rm -f "${MOUNT_DIR}/test"
ls "${MOUNT_DIR}"

echo Add persistent mount
printf "\nUUID=$(sudo blkid -s UUID -o value "${DEVICE_PARTITION}") ${MOUNT_DIR} ext4 defaults 0 0\n" \
| sudo tee -a "/etc/fstab"
## sudo nano "/etc/fstab"
```

## Fix disk errors

```shell script
GRUB menu
Advanced options for Ubuntu
Recovery mode

sudo umount \
    --force \
    --lazy \
    "/data1" \
    "/data2"

## Check not mounted FS
sudo fsck -M -C -v -y

## Check all FS
sudo fsck -A -C -v -y

## Check all FS except root mount
sudo fsck -A -C -v -y

sudo fsck -y /dev/sdx

dumpe2fs /dev/sdx | grep superblock

sudo fsck -b 32768 /dev/sdx
```

## Check LVM 

```shell script
lvscan

fsck /dev/ubuntu-vg/ubuntu-lv
```

## Extend LVM 

```shell script
sudo lsblk -p

sudo lvs

sudo lvextend -l +100%FREE "/dev/mapper/ubuntu--vg-ubuntu--lv"

resize2fs "/dev/mapper/ubuntu--vg-ubuntu--lv"

df -h

sudo lsblk -p
```

## Get disk temperature 

```shell script
sudo apt-get install -y smartmontools

sudo smartctl -a /dev/sda
```

## Enable auto disk check

```shell script
sudo touch /forcefsck
```

## Disable auto disk check

```shell script
sudo nano /etc/fstab
```
```text
## Set Dump and Pass to 0/0
uuid=<uuid> / ext4 defaults 0 0
```

## Clone a disk

### Clone a disk as-is into the disk of the same (or larger) size

```shell script
export SOURCE_LETTER="c"
export TARGET_LETTER="b"

sudo dd \
    if=/dev/sd${SOURCE_LETTER}\
    of=/dev/sd${TARGET_LETTER} \
    conv=noerror,sync \
    status=progress \
    2>&1
```

### Create disk image

```shell script
sudo dd \
    if=/dev/sdb \
    of=/tmp/raw.img \
    conv=noerror,sync \
    status=progress \
    2>&1
```

### View disk image contains

```shell script
sudo losetup \
    --find \
    --partscan \
    --show \
    "/tmp/raw.img"

sudo mount /dev/loop0p1 /mnt
ls /mnt
```
