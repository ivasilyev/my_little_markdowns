# Tweak a `*.vmx` file directly

```
# Add disk UUUD support
disk.EnableUUID = "TRUE"

# Fix losing signal of the host's keyboard and mouse
keyboard.allowBothIRQs = "FALSE"

# Fix losing signal of the host's mouse
mks.gamingMouse.policy = "gaming"

# Or fix too fast mouse
mks.gamingMouse.policy = "absolute"

# Enable hardware video acceleration
mks.dx11.allowUnsupportedDevices = "TRUE"
mks.enable3d = "TRUE"
mks.enableDX12 = "TRUE"
mks.enableDX12Renderer = "FALSE"
mks.enableGLRenderer = "TRUE"
mks.enableVulkanRenderer = "FALSE"
mks.gl.allowBlacklistedDrivers = "TRUE"
mks.gl.checkHostDriver = "FALSE"
mks.ignoreHostDriverVersion = "TRUE"
mks.vk.allowUnsupportedDevices = "TRUE"
mks.vk.forceDevice = "TRUE"
```

# Change NIC type

In `<vm>.vmx`, change `ethernet#.virtualDev = ""`

Common possible values are `e1000`, `e1000e`, `vmxnet3`.

The information is available elsewhere, e.g.:
* https://rickardnobel.se/vmxnet3-vs-e1000e-and-e1000-part-1/
* https://kb.vmware.com/s/article/1001805

The best practice from VMware is to use the 10Gbit `vmxnet3` Virtual NIC 
unless there is a specific driver or compatibility reason where it cannot be used.
