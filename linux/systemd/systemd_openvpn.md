# OpenVPN client setup

```shell script
sudo apt-get update -y && \
sudo apt-get install -y openvpn

sudo nano /etc/openvpn/client.conf
```

```shell script
export CLIENT_SERVICE="openvpn@client.service"

sudo systemctl enable "${CLIENT_SERVICE}"
sudo systemctl restart "${CLIENT_SERVICE}"
sudo systemctl status "${CLIENT_SERVICE}"
```
