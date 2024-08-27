# Short reference about conversion into VMware VM from a virtual or physical  machine 

## From an entire disk containing physical machine

### Create raw disk image

- Find the block device (e.g. `/dev/sdb`):

```shell script
# For Windows use Cygwin

ls -la "/dev/"
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
echo "Convert RAW to VMDK"
"${PROGRAMFILES}/Oracle/VirtualBox/VBoxManage.exe" \
    convertfromraw \
    "disk.img" \
    "disk.vmdk" \
    --format vmdk
mkdir \
    --parent \
    --verbose \
    "split"
echo "Convert 'monolithic sparse' VMDK into 'split sparse' VMDK"
"${PROGRAMFILES} (x86)/VMware/VMware Workstation/vmware-vdiskmanager.exe" \
    -r "disk.vmdk" \
    -t 1 \
    "split/disk.vmdk"
```

### Create VMware VM

- Invoke VMware VM creation wizard
- Select advanced configuration
- Select `I will install the operating system later`
- Select `SATA` as virtual disk type
- Select the existing `VMDK` file
