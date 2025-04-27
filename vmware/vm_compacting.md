# Virtual machine disk compression

## Guest

```shell script
# Unmount all shares beforehand

echo "Install software"
sudo apt-get update -y && sudo apt-get install -y open-vm-tools

echo "Defragment root" && \
sudo e4defrag / >/dev/null 2>&1 && \
echo "Zero-fill all unused space" && \
dd if=/dev/zero of=/tmp/wipefile bs=1M && \
sync && \
rm -f /tmp/wipefile && \
echo "Run the shrink operation" && \
sudo vmware-toolbox-cmd disk shrinkonly
```

## Host

```text
- VM
- Settings
- Hard disk
- Compact
```
