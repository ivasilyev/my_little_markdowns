# Virtual machine disk compression

## Guest

```shell script
echo "Install software"
sudo apt-get update -y && sudo apt-get install -y open-vm-tools

export TOOL_SCRIPT="/usr/local/bin/shrink_vm.sh"

cat <<EOF | sudo tee "${TOOL_SCRIPT}"
echo "Unmount all shares beforehand"
sudo umount -a

echo "Defragment root"
sudo e4defrag / >/dev/null 2>&1

echo "Zero-fill all unused space"
dd if=/dev/zero of=/tmp/wipefile bs=1M
sync
rm -f /tmp/wipefile

echo "Run the shrink operation"
sudo vmware-toolbox-cmd disk shrinkonly

echo "Reboot"
sudo shutdown -r now
EOF

# Schedule script run at 00:00 on every Friday
echo "0 0 * * 0 /usr/bin/env bash '${TOOL_SCRIPT}'"

sudo crontab -e
```

## Host

```text
- VM
- Settings
- Hard disk
- Compact
```
