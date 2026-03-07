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

# Enable hardware video acceleration in the case of
# 3D acceleration will be disabled for VMs as DirectX 11.1 is not supported by the host
mks.dx11.allowUnsupportedDevices = "TRUE"
mks.dx12.allowUnsupportedDevices = "TRUE"
mks.enable3d = "TRUE"
mks.enableDX11 = "FALSE"
mks.enableDX11Renderer = "FALSE"
mks.enableDX12 = "FALSE"
mks.enableDX12Renderer = "FALSE"
mks.enableGLRenderer = "TRUE"
mks.enableMTLRenderer = "FALSE"
mks.enableVulkanRenderer = "FALSE"
mks.gl.allowUnsupportedDrivers="TRUE"
mks.gl.allowBlacklistedDrivers = "TRUE"
mks.gl.checkHostDriver = "FALSE"
mks.ignoreHostDriverVersion = "TRUE"
mks.vk.allowUnsupportedDevices = "TRUE"
mks.vk.forceDevice = "TRUE"
pref.someLegacyOption = "TRUE"
svga.autodetect = "FALSE"
svga.graphicsMemoryKB = "1048576"
svga.noDrivers = "TRUE"
```

# Change NIC type

In `<vm>.vmx`, change `ethernet#.virtualDev = ""`

Common possible values are `e1000`, `e1000e`, `vmxnet3`.

The information is available elsewhere, e.g.:
* https://rickardnobel.se/vmxnet3-vs-e1000e-and-e1000-part-1/
* https://kb.vmware.com/s/article/1001805

The best practice from VMware is to use the 10Gbit `vmxnet3` Virtual NIC 
unless there is a specific driver or compatibility reason where it cannot be used.
