# Install SSH & allow it via UFW

```shell script
sudo apt-get update -y && \
sudo apt-get install -y openssh-server && \
sudo systemctl enable ssh.service && \
sudo systemctl restart ssh.service && \
sudo ufw allow proto tcp to 0.0.0.0/0 port 22 comment "OpenSSH server listen port" && \
sudo ufw --force disable && \
sudo ufw --force enable && \
sudo ufw status verbose

# Run by a regular user!
echo Create SSH identities
ssh-keygen \
    -N "" \
    -t ed25519 \
    -b 1024 \
    -f /home/user/.ssh/id_ed25519 && \
ssh-keygen \
    -N "" \
    -t rsa \
    -b 1024 \
    -f /home/user/.ssh/id_rsa && \
chmod -v 600 ~/.ssh/authorized_keys && \
chmod -v 700 ~/.ssh

# Add remote hosts connection
# nano ~/.ssh_config
# Edit remote keys to be accepted if required
# nano ~/.ssh/authorized_keys
# chmod -v 600 ~/.ssh/authorized_keys
# ssh-copy-id -i ~/.ssh/id_ed25519.pub username@hostname
```

# Install RDP

```shell script
sudo apt-get install \
    --yes \
    avahi-daemon \
    samba \
    samba-common \
    winbind \
    wsdd \
    xrdp

sudo systemctl enable xrdp
sudo systemctl restart xrdp
sudo systemctl set-default multi-user.target
sudo reboot
```

# Install AnyDesk

## Manage dependencies

```shell script
cd "/tmp"
echo Install AnyDesk
sudo apt-get install \
    --fix-broken \
    --yes \
    libgtkglext1 \
    libpango-1.0-0

wget http://ftp.us.debian.org/debian/pool/main/p/pangox-compat/libpangox-1.0-0_0.0.2-5.1_amd64.deb
sudo dpkg -i libpangox-1.0-0_0.0.2-5.1_amd64.deb
```

## The legacy version (recommended)

```shell script
sudo systemctl disable anydesk.service
sudo systemctl stop anydesk.service
sudo pkill anydesk
sudo apt remove \
    --purge \
    --yes anydesk

cd "/tmp"
# export URL="https://download.anydesk.com/linux/anydesk_6.3.2-1_amd64.deb"
export URL="https://arquivos.blogdainformatica.com.br/redes-e-internet/anydesk/anydesk_5.1.2-1_amd64.deb"
curl -fsSL \
    "${URL}" \
    -o "anydesk.deb"
sudo dpkg \
    --ignore-depends=libpango1.0-0 \
    --install "anydesk.deb"
sudo systemctl enable anydesk.service
sudo systemctl restart anydesk.service

# Comment/remove the anydesk line
sudo nano /etc/apt/sources.list.d/anydesk-stable.list
sudo apt-get update
```

## The modern version (much slower)

```shell script
echo Add AnyDesk repository key to Trusted software providers list
sudo wget -qO - https://keys.anydesk.com/repos/DEB-GPG-KEY | sudo apt-key add -

echo Add AnyDesk repository
echo "deb http://deb.anydesk.com/ all main" | sudo tee /etc/apt/sources.list.d/anydesk-stable.list
# sudo echo "deb http://ftp.debian.org/debian unstable main contrib non-free" >> /etc/apt/sources.list.d/debian.list

echo Update APT cache
sudo apt-get update -y

echo Install AnyDesk
sudo apt-get install \
    --fix-broken \
    --yes \
    anydesk

echo Start AnyDesk
sudo systemctl enable anydesk
sudo systemctl restart anydesk
```
