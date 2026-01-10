# Linux network tweaks

## Enable Ubuntu TCP BBR to improve network performance

### Check available and currently using congestion control algorithms

```
sudo sysctl net.ipv4.tcp_available_congestion_control
sudo sysctl net.ipv4.tcp_congestion_control
```

### Manage kernel

```
cat <<EOF | sudo tee -a "/etc/sysctl.conf"

net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF

# sudo nano "/etc/sysctl.conf"
```

### Reload and check `sysctl` configurations

```
sudo sysctl -p

sudo sysctl net.ipv4.tcp_available_congestion_control \
| grep -q 'bbr' \
&& echo '1 Yes'

sudo sysctl net.ipv4.tcp_congestion_control \
| grep -q 'bbr' \
&& echo '2 Yes'

sudo sysctl net.core.default_qdisc \
| grep -q 'fq' \
&& echo '3 Yes'

sudo lsmod \
| grep bbr \
| grep -q 'tcp_bbr' \
&& echo '4 Yes'
```
