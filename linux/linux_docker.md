# Install Docker

```shell script
echo Set up the repository && \
sudo apt-get update -y && \
sudo apt-get install \
    --yes \
    ca-certificates\
    curl \
    gnupg && \
echo Add Docker’s official GPG key && \
sudo install -m 0755 -d /etc/apt/keyrings && \
curl \
    -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg \
    --dearmor \
    -o /etc/apt/keyrings/docker.gpg && \
sudo chmod a+r /etc/apt/keyrings/docker.gpg && \
echo Add the repository to APT sources && \
echo "
    deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && \
        echo "$(
            lsb_release \
                --codename \
                --short
        )"
    ) stable
" \
| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null && \
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
# sudo usermod -aG docker "$(whoami)"

sudo groupadd -f docker && \
sudo usermod -aG docker user && \
sudo newgrp docker && \
sudo shutdown -r now
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

docker container prune
docker image prune
docker volume prune
docker network prune
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

sudo rm -rf /var/lib/docker
```
