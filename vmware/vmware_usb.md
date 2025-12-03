# Automatically connect USB devices at virtual machine power on

## Identify and Obtain the USB Device's Vendor ID and Product ID

### Linux host

After you connect the USB device, you can find the vendor and product ID in the `/proc/bus/usb/devices` file.

For example:

```
more /proc/bus/usb/devices
T: Bus=01 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#= 3 Spd=1.5 MxCh= 0
D: Ver= 2.00 Cls=ff(vend.) Sub=00 Prot=00 MxPS= 8 #Cfgs= 1
P: Vendor=0529 ProdID=0001 Rev= 2.15
S: Manufacturer=AKS
S: Product=HASP HL 2.15
```

Here, 529 is the vendor ID and 1 is the product ID for the HASP HL device.

### Windows host

You may find the vendor and product IDs via `devmgmt.msc` or in the registry.

Search for your USB device's name or brand (`regedit` is also applicable):

`reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\`

This example output is a result of searching for a Cruzer Mini USB key:

```text
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\Vid_0781&Pid_7101\00526174
HardwareID: USB\Vid_0781&Pid_7101&Rev_0102
USB\Vid_0781&Pid_7101
LocationInformation: Cruzer Mini
Mfg: Compatible USB storage device
```

Here, the vendor ID is `781` and the product ID is `7101`.

## Edit the virtual machine's configuration file

The configuration (`.vmx`) file is located in the same directory in which the virtual machine was created. 
Ensure that the virtual machine is powered off before you edit this file.

To autoconnect the HASP HL device from the previous example, add this line in the .vmx file of the virtual machine:

`usb.autoConnect.device0 = "0x529:0x1"`

Or, for the Cruzer Mini device, add:

`usb.autoConnect.device0 = "0x781:0x7101"`

Note: Prepend 0x to each value — it must be in hex format.

You can specify multiple devices for autoconnect, provided that there are not more than two USB devices available to 
the host at the same time. You can see multiple entries for autoConnect in this example. An ellipsis [...] indicates
 the omission of an actual vendor or product ID. You need to include a specific value, as shown for device0 and device1.

```text
usb.autoConnect.device0 = "0x529:0x1"
usb.autoConnect.device1 = "0x781:0x7101"
usb.autoConnect.device2 = ....
usb.autoConnect.device3 = ....
```

You can use the auto clean option in usb.autoconnect:

- `autoclean:1` - autoconnect if a device matches the pattern, removed if the VM is powered on and no device matches 
the pattern, removed if disconnected through the UI.
- `autoclean:0`, no autoclean - autoconnect if a device matches the pattern, not removed if the VM is powered on and no 
device matches the pattern, not removed if disconnected through UI. Although the autoconnect entries are not removed, 
the device will not autoconnect again after the user disconnects the device using the UI for that session of the VM. 
The same device will autoconnect when the VM is restarted or when the device is physically unplugged and replugged into 
the same port on the host.

For example, you may enter this option at the end of the autoconnect line:
`usb.autoConnect.device0 = "vid:0x781 pid:0x7101 autoclean:1"`

For the USB3 devices use:

`usb_xhci.autoconnect.device0 = "vid:0x0BDA pid:B812"`

This will also connect all identical device to the VM.
