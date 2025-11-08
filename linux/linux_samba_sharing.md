# Mount remote Samba share to Linux client

## Supply remote share credentials

```shell script
# Run under regular user
echo Install software
sudo apt-get update -y && sudo apt-get install -y cifs-utils smbclient

unset HISTFILE
echo Export variables
#
export REMOTE_UN=""
export REMOTE_PASSWD=""
export REMOTE_HOST=""
export LOCAL_DIR=""
export REMOTE_DIR=""
#
export FULL_REMOTE_DIR="${REMOTE_HOST}/${REMOTE_DIR}"
export LOCAL_CFG="/etc/samba/${REMOTE_HOST}.smbclient"
clear

echo Check remote host availability
ping -c 4 "${REMOTE_HOST}"

echo Create Samba client credentials
cat <<EOF | sudo tee "${LOCAL_CFG}"
# //${REMOTE_HOST}
username=${REMOTE_UN}
password=${REMOTE_PASSWD}
EOF
# sudo nano "${LOCAL_CFG}"

echo Or use a function
create_samba_credentials() {
    export REMOTE_HOST="${1}"
    export REMOTE_UN="${2}"
    export REMOTE_PASSWD="${3}"
    export LOCAL_CFG="/etc/samba/${REMOTE_HOST}.smbclient"
    
    printf "# %s\nusername=%s\npassword=%s\n" "${REMOTE_HOST}" "${REMOTE_UN}" "${REMOTE_PASSWD}" | sudo tee "${LOCAL_CFG}" 
}

echo Create Samba client mount point
sudo mkdir -pv "${LOCAL_DIR}"
sudo chmod -Rv 777 "${LOCAL_DIR}"

echo Set Samba client mount point as persistent
printf "\n#${REMOTE_HOST}\n//${FULL_REMOTE_DIR} ${LOCAL_DIR} cifs rw,_netdev,credentials=${LOCAL_CFG},iocharset=utf8,uid=$(id -u),gid=$(id -g) 0 0\n" | sudo tee -a "/etc/fstab"
# sudo nano "/etc/fstab"
clear

# Reboot
sudo shutdown -r now
```

## Mount shares

```shell script
sudo umount "${LOCAL_DIR}"

sudo mount \
    --options credentials="${LOCAL_CFG}" \
    --types cifs \
    --verbose \
    "${FULL_REMOTE_DIR}" \
    "${LOCAL_DIR}"
    
sudo mount \
    --all \
    --options remount \
    --verbose

sudo systemctl enable systemd-networkd-wait-online.service networkd-dispatcher.service systemd-networkd.service
sudo systemctl restart systemd-networkd-wait-online.service networkd-dispatcher.service systemd-networkd.service
```


# Create Samba server share

```shell script
echo Export variables
export UN=""
export PASSWD=""
export SHARE_NAME=""
export SHARE_DIR=""

export TOOL_CFG_FILE="/etc/samba/smb.conf"
export TOOL_SERVICES="nmbd.service smbd.service wsdd.service"
unset HISTFILE
clear

echo Uninstall software
apt-get purge \
    --yes \
    samba \
    samba-common \
    winbind

echo Install software
sudo apt-get update -y && \
sudo apt-get install \
    --yes \
    avahi-daemon \
    cifs-utils \
    nfs-common \
    psmisc \
    samba \
    samba-common \
    smbclient \
    wsdd

echo Create special user \'${UN}\' with read-only access to Samba
sudo userdel "${UN}"
sudo useradd \
    --no-create-home \
    --shell "$(/usr/bin/env false)" \
    --no-user-group \
    "${UN}"
echo "${UN}:${PASSWD}" | chpasswd

echo Add Samba password for user with read-only permissions
echo -ne "${PASSWD}\n${PASSWD}\n" | smbpasswd -a -s "${UN}"

echo "(Optional) Add Samba password for user with read-write permissions"
sudo smbpasswd -a "$(whoami)"

# bash_functions required
echo Configure Web Service Discovery host and client daemon.
cat <<EOF | sudo tee /etc/wsdd.conf
WSDD_PARAMS="--shortlog --interface=$(get_default_nic) --hostname=$(hostname) --workgroup=$(grep -i '^\s*workgroup\s*=' "${TOOL_CFG_FILE}" | cut -f2 -d= | tr -d '[:blank:]') "
EOF
# sudo nano /etc/wsdd.conf

echo Forward SMB ports via firewall
sudo ufw allow to 0.0.0.0/0 port 137 comment "NetBIOS Name Service (WINS) ports"
sudo ufw allow to 0.0.0.0/0 port 138 comment "NetBIOS Datagram Service ports"
sudo ufw allow to 0.0.0.0/0 port 139 comment "TCP NetBIOS Session (TCP), Windows File and Printer Sharing port"
sudo ufw allow to 0.0.0.0/0 port 445 comment "Microsoft Directory Services ports"
# Then follow `linux_firewall.md`

echo Backup Samba configuration
sudo cp \
    "${TOOL_CFG_FILE}" \
    "${TOOL_CFG_FILE}.bak"

echo Configure Samba
grep -v '^ *#\|^ *$' "${TOOL_CFG_FILE}" \
| sudo tee "${TOOL_CFG_FILE}"

echo Create directory \'${SHARE_DIR}\' for share \'${SHARE_NAME}\'
sudo mkdir -p "${SHARE_DIR}"
sudo chmod -R 777 "${SHARE_DIR}"
sudo umount -a -t cifs -l
# sudo chown -R "${UN}" "${SHARE_DIR}"


cat <<EOF | sudo tee ${TOOL_CFG_FILE} 
[global]
aio read size = 16384
aio write size = 16384
dns proxy = no
log file = /var/log/samba/log.%m
logging = file
map to guest = never
max log size = 1000
min receivefile size = 16384
obey pam restrictions = yes
pam password change = yes
panic action = /usr/share/samba/panic-action %d
passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
passwd program = /usr/bin/passwd %u
read raw = Yes
restrict anonymous = 2
server role = standalone server
server string = %h server (Samba, Ubuntu)
socket options = TCP_NODELAY IPTOS_LOWDELAY SO_RCVBUF=131072 SO_SNDBUF=131072
unix password sync = yes
use sendfile = true
usershare allow guests = no
workgroup = WORKGROUP
write raw = Yes

[printers]
available = yes
browseable = no
comment = All Printers
create mask = 0700
guest ok = no
path = /var/spool/samba
printable = yes
read only = yes

[print$]
available = yes
browseable = yes
comment = Printer Drivers
guest ok = no
path = /var/lib/samba/printers
read only = yes

[${SHARE_NAME}]
available = yes
browseable = yes
comment = ${SHARE_NAME}
create mask = 0775
directory mask = 0775
force create mode = 0775
force directory mode = 0775
force user = ${UN}
guest ok = no
path = ${SHARE_DIR}
read list = ${UN}
read only = yes
write list = root $(whoami)
writeable = no

EOF
# sudo nano "${TOOL_CFG_FILE}"



echo Restart services
# Quotes are not required here
sudo systemctl daemon-reload
sudo systemctl enable ${TOOL_SERVICES}
sudo systemctl restart ${TOOL_SERVICES}
sudo systemctl status ${TOOL_SERVICES}

echo Test Samba share connection
testparm
smbclient \
    -U "${SHARE_NAME}" \
    -L localhost
```

# Disable SELINUX

```shell script
sudo nano /etc/selinux/config
```
