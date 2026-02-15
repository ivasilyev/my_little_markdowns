# UFW (Ubuntu’s Uncomplicated firewall)

## Installation

```shell script
sudo apt-get install -y ufw
```

## Forward port via firewall

```shell script
sudo ufw allow proto tcp to 0.0.0.0/0 port 22 comment "OpenSSH server listen port"

sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose
```

## View rule numbers

```shell script
sudo ufw status numbered
```

## Delete the rule with the number `999`

```shell script
sudo ufw --force delete 999
```

## Delete the rule for the port `65535`

```shell script
sudo ufw --force delete allow 65535
```

## NAT the connections from the external interface to the internal

### Allow forwarded packets by default

```shell script
sudo sed \
    -i 's|^DEFAULT_FORWARD_POLICY="DROP"|# DEFAULT_FORWARD_POLICY="DROP"\nDEFAULT_FORWARD_POLICY="ACCEPT"|g'\
    "/etc/default/ufw"

# sudo nano /etc/default/ufw

sudo nano /etc/ufw/sysctl.conf
```

```text
net.ipv4.ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1
```

```shell script
sudo sysctl -p
```

### Add NAT to the UFW configuration 

```shell script
# Requires bash functions

export TOOL_NAME=""
export NIC="$(get_default_nic)"
export EXT_IP="$(get_external_ip)"
export IP_CHUNK="."

printf "
# Start ${TOOL_NAME} rules
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic for ${TOOL_NAME} to ${NIC}
-A POSTROUTING -s ${IP_CHUNK}0/24 -o ${NIC} -j MASQUERADE
COMMIT
# End ${TOOL_NAME} rules
"

# Paste the contents into the file
sudo nano /etc/ufw/before.rules
```
```shell script
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward

# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Forward traffic through eth0 - Change to match you out-interface
-A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# Don't delete the 'COMMIT' line or these nat table rules won't be processed
COMMIT

# Don't delete these required lines, otherwise there will be errors
COMMIT

*filter
. . .
```

## Port forwarding

```shell script
sudo nano /etc/ufw/before.rules
```
```
# NAT table rules
*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]

# Port Forwardings
-A PREROUTING -i eth0 -p tcp --dport 22 -j DNAT --to-destination 192.168.1.10

# Don't delete these required lines, otherwise there will be errors
COMMIT
```



### (Deprecated) `iptables`

```shell script
iptables \
    -4 \
    -A INPUT \
    -p tcp \
    --dport ${TOOL_PORT_1} \
    -m comment \
    --comment \
    "Server listen port" \
    -j ACCEPT
iptables \
    -4 \
    -A INPUT \
    -p udp \
    --dport ${TOOL_PORT_1} \
    -m comment \
    --comment \
    "Server listen port" \
    -j ACCEPT
```
