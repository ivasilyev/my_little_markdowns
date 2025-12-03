# VMware basic instructions

## Download and install VMware tools to VM 

[VMware website](https://packages.vmware.com/tools/)

## Edit - Preferences

### Display

- (off) Autofit Window
- (off) Autofit Guest
- Center Guest
- (on) Hardware acceleration for remote virtual machine

### USB

- Connect the device to the host

### Updates

- (off) Check for product updates at startup
- (off) Check for software components as needed
- (off) Automatically update VMware Tools on a virtual machine

### Feedback

- (off) Join the VMware Customer Experience Improvement Program

### Memory

- Fit all virtual machine memory into reserved host RAM

## Create new VM

* Memory: 4GB
* Processors: 
  * Number of processors: 1
  * Number of cores per processors: 4
  * (on) Virtualize Intel VT-x/EPT or AMD-v/RVI
  * (on) Virtualize IOMMU (IO memory management unit)
* Network Adapter: bridged
* USB Controller
  * USB comptibility: USB 3.1
* Sound Card: Use default host sound card
* Display
  * 3D graphics: (on) Accelerate 3D graphics
  * Display scaling:
    * (off) Automatically adjust user interface size in the virtual machine
    * Stretch mode: Automatically adjust user interface in the VM

## Add extra NIC

* Select Window > Virtual Machine Library.
* Select a virtual machine in the Virtual Machine Library window and click Settings.
* Click Add Device.
* Click Network Adapter.
* Click Add.
* Either select a network configuration from the list or, if you have Fusion Pro, click Configure below the list to 
create a new network.

## Increase USB compatibility

* VM
* Edit virtual machine settings
* Hardware
* USB Controller: 3.1

## Fix for the network bridge mode not working on Windows 10 host

- Be sure your vm is stopped.
- Run the VMWare Virtual Network Editor
(click start and search for Virtual Network Editor)
- Run it as administrator (or click the button at the bottom of the screen that says, "change settings." 
VMNet0 will dislpay when running as administrator. Otherwise, it will not be visible)
- Highlight VMNet0 and click on "Automatic Settings"
- You will see a list of adapters. De-select all but the physical network card. (When I set up up with player, 
I had selected only the 1.  After install of workstation, all of the items were checked.)
- Click "OK"
- Click "OK"
- Start the VM and test. 

## Fix for `virtual device serial 0:1 will start disconnected`

- Open `cmd`
- `notepad "%PROGRAMDATA%\VMware\VMware Workstation\settings.ini"`
- Change `printers.enabled = "FALSE"` into `printers.enabled = "TRUE"`
- Save changes
