# Short reference about conversion into VMware VM from a virtual or physical  machine 

Use **Cygwin** or **Git Bash**.

## From an entire disk containing physical machine

### Create raw disk image

- Find the block device (e.g. `/dev/sdb`):

```shell script
# For Windows use Cygwin

ls -la "/dev/"

for i in /dev/s* ; do printf "${i}\t$(cygpath -w "${i}")\n" ; done
```

- Dump the raw disk image (e.g. `disk.img`) using the `dd` command from `linux disk_utils`.

### Convert physical drive image into `VDI` file

VMware seemingly does not support direct import from a RAW VM disk format.  
It is recommended to convert the disk image into the `VDI`, Virtual Disk Image, 
the VirtualBox Native format, import it into new VirtualBox VM, convert this VM
into an `OVA` (Open Virtual Appliance) file that may be imported with VMware.

```shell script
"${PROGRAMFILES}/Oracle/VirtualBox/VBoxManage.exe" \
    convertfromraw \
    "disk.img" \
    "disk.vdi" \
    --format vdi
```

### Import `VDI` file into VirtualBox VM

- Create new virtual machine using the VDI file as existing SATA drive.
- Power off the virtual machine in VirtualBox. If the virtual machine is in a suspended state, 
power on the virtual machine and then shut it down.

### Export VirtualBox VM into OVA

- Click File > Export Appliance.
- Select one or more VMs to export, and click Next.
- Select a Format & provide a location to store the OVA v1.0 file.
- Follow the onscreen instructions to make the necessary changes.
- Uncheck Sound card and Ethernet controller.
- Click Export to begin the export process.

## From OVA VM archive file

### VMware Fusion

- Import VM as Open Virtualization Format Virtual Machine

### VMware Workstation and Player

– Open new VM as an Open Virtualization Format Virtual Machine

## From raw disk image via VMDK

The approach of direct conversion of a raw disk image 
into VMware `VMDK` (Virtual Machine Disk) format is less space-efficient 
yet more fast and works for almost all cases of importing Linux VMs.

### Create VMDK file

```shell script
echo "Export variables"
#
export DEV_LETTER="b"
export VM_BASENAME="my_drive"
export IMG_FILE="D:/${VM_BASENAME}/${VM_BASENAME}.img"
# It is recommended to use different disks during the conversion for an optimal I/O
export VMDK_MONO_FILE="E:/${VM_BASENAME}/${VM_BASENAME}.vmdk"
export VMDK_SPLIT_FILE="D:/${VM_BASENAME}/${VM_BASENAME}.vmdk"
#

echo "Convert RAW to VMDK"
"${PROGRAMFILES}/Oracle/VirtualBox/VBoxManage.exe" \
    convertfromraw \
    "${IMG_FILE}" \
    "${VMDK_MONO_FILE}" \
    --format vmdk
mkdir \
    --parent \
    --verbose \
    "$(dirname "${VMDK_MONO_FILE}")" \
    "$(dirname "${VMDK_SPLIT_FILE}")"
echo "Convert 'monolithic sparse' VMDK into 'split sparse' VMDK"
"${PROGRAMFILES} (x86)/VMware/VMware Workstation/vmware-vdiskmanager.exe" \
    -r "${VMDK_MONO_FILE}" \
    -t 1 \
    "${VMDK_SPLIT_FILE}"
```

### Rename VM disk

```shell script
"${PROGRAMFILES} (x86)/VMware/VMware Workstation/vmware-vdiskmanager.exe" \
    -n "old.vmdk" \
    "new.vmdk"
```

### Create VMware VM

- Invoke VMware VM creation wizard
- Select advanced configuration
- Select `I will install the operating system later`
- Select `SATA` as virtual disk type
- Select the existing `VMDK` file
