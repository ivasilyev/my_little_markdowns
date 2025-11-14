# Install Docker

```shell script
echo Set up the repository && \
sudo apt-get update  && \
sudo apt-get install ca-certificates curl && \
sudo install -m 0755 -d /etc/apt/keyrings && \
sudo curl -fsSL \
  'https://download.docker.com/linux/ubuntu/gpg' \
  -o '/etc/apt/keyrings/docker.asc' && \
sudo chmod a+r '/etc/apt/keyrings/docker.asc' && \
echo Add the repository to sources && \
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt-get update && \
sudo systemctl daemon-reload && \
sudo apt-get update -y && \
echo Install the Docker engine && \
sudo apt-get install \
    --yes \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

# Run undo ordinary user (use `logout` if required)

```shell script
sudo groupadd -f docker && \
sudo usermod -aG docker "$(whoami)" && \
sudo newgrp docker && \
sudo shutdown -r now
exit
```

# Test Docker

```shell script
export IMG="hello-world:latest"
docker pull "${IMG}"
docker run \
    --interactive \
    --rm \
    --tty \
    "${IMG}"
```

# Manage cron jobs

```shell script
echo Add cron job to purge all images every reboot
echo @reboot \"$(which docker)\" system prune --all --force \> /dev/null 2\>\&1

echo Add cron job to pull image every day at 12AM
echo "0 0 * * * \"$(which docker)\" pull \"${IMG}\""

sudo crontab -e

sudo crontab -l
```

# Uninstall Docker

```shell script
# Cleanup
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)

docker container prune --force
docker image prune --force
docker volume prune --force
docker network prune --force
docker system prune --all --volumes --force

# Remove
sudo apt-get remove \
    --purge \
    --yes \
    containerd.io \
    docker-compose \
    docker-compose-v2 \
    docker-doc \
    docker.io \
    docker-ce \
    docker-ce-cli \
    docker-buildx-plugin \
    docker-compose-plugin \
    podman-docker \
    runc

# Or
sudo apt remove $(
  dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc \
  | cut -f1
)

sudo rm -rf /var/lib/docker
```
