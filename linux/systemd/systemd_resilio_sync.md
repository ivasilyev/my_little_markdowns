

sudo apt-get update -y
sudo apt-get install -y \
    curl \
    git \
    wget \
    open-vm-tools-desktop \
    openssh-server

sudo ufw allow proto tcp to 0.0.0.0/0 port 22 comment "OpenSSH server listen port"

sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo "deb http://linux-packages.resilio.com/resilio-sync/deb resilio-sync non-free" | sudo tee /etc/apt/sources.list.d/resilio-sync.list

wget -qO- https://linux-packages.resilio.com/resilio-sync/key.asc | sudo tee /etc/apt/trusted.gpg.d/resilio-sync.asc > /dev/null 2>&1

sudo apt-get update -y
sudo apt-get install resilio-sync
# Unable to fetch

curl \
    https://download-cdn.resilio.com/3.1.2.1076/linux/amd64/0/resilio-sync-amd64.deb \
    -o /tmp/rsync.deb
# Unable to fetch

scp \
    "$(
        cygpath "${USERPROFILE}/Downloads/resilio-sync-amd64.deb"
    )" \
    user@10.48.15.124:/tmp/rsync.deb

sudo dpkg --install /tmp/rsync.deb



cat <<EOF | sudo tee /etc/resilio-sync/config.json
{
    "storage_path" : "/var/lib/resilio-sync/",
    "pid_file" : "/var/run/resilio-sync/sync.pid",

    "webui" :
    {
        "force_https": true,
        "listen" : "0.0.0.0:8888"
    }
}
EOF


sudo systemctl enable resilio-sync.service
sudo systemctl restart resilio-sync.service


sudo lsof -i -P -n | grep rslsync
sudo ufw allow to 0.0.0.0/0 port 42643 comment "rslsync P2P listen port"
sudo ufw allow proto tcp to 0.0.0.0/0 port 8888 comment "rslsync server listen port"
sudo ufw allow proto tcp to 0.0.0.0/0 port 57938 comment "OpenSSH server listen port"

sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose
