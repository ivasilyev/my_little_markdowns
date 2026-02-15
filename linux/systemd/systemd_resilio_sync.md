# Deploy Resilio Sync

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="rslsync"
export USER_NAME="${TOOL_NAME}_user"
export USER_PASSWORD=""
#
export TOOL_PORT=55555
export TOOL_WEB_PORT=8888
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_DATA_DIR="${TOOL_DIR}data"
export TOOL_CFG_DIR="${TOOL_DIR}config"
export TOOL_DOWNLOAD_DIR="/tmp/dl"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
# Resilio Sync began requiring registration for the free version starting with version 3.0.
export IMG="ghcr.io/linuxserver/resilio-sync:2.8.1"

echo Create directories
sudo rm \
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}"
sudo mkdir \
    --parent \
    --mode 0700 \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "$(whoami)")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
```

## Inspect & debug Docker image if it does contain shell

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env "PUID=$(id --user "$(whoami)")" \
    --env "PGID=$(id --group "$(whoami)")" \
    --env "TZ=Etc/UTC" \
    --env TOOL_CFG_DIR="${TOOL_CFG_DIR}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}/tcp" \
    --publish "${TOOL_PORT}:${TOOL_PORT}/udp" \
    --publish "${TOOL_WEB_PORT}:${TOOL_WEB_PORT}" \
    --rm \
    --tty \
    --volume "${TOOL_CFG_DIR}:/config" \
    --volume "${TOOL_DATA_DIR}:/sync" \
    --volume "${TOOL_DOWNLOAD_DIR}:/downloads" \
    "${IMG}"

/usr/bin/rslsync --help
```

## Start tool

```shell script
echo Create ${TOOL_NAME} routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "PUID=$(id --user "$(whoami)")" \\
    --env "PGID=$(id --group "$(whoami)")" \\
    --env "TZ=Etc/UTC" \\
    --env TOOL_CFG_DIR="${TOOL_CFG_DIR}" \\
    --name "${TOOL_NAME}" \\
    --publish "${TOOL_PORT}:${TOOL_PORT}/tcp" \\
    --publish "${TOOL_PORT}:${TOOL_PORT}/udp" \\
    --publish "${TOOL_WEB_PORT}:${TOOL_WEB_PORT}" \\
    --rm \\
    --volume "${TOOL_CFG_DIR}:/config" \\
    --volume "${TOOL_DATA_DIR}:/sync" \\
    --volume "${TOOL_DOWNLOAD_DIR}:/downloads" \\
    "\${IMG}"
EOF

sudo chmod a+x "${TOOL_SCRIPT}"
# nano "${TOOL_SCRIPT}"



echo Create ${TOOL_NAME} system service
cat <<EOF | sudo tee "${TOOL_SERVICE}"
[Unit]
Description=${TOOL_NAME}
Documentation=https://google.com
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=root
ExecReload=/usr/bin/env docker stop "${TOOL_NAME}"; /usr/bin/env kill -s SIGTERM \$MAINPID
ExecStart=/usr/bin/env bash "${TOOL_SCRIPT}"
SyslogIdentifier=${TOOL_NAME}
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# nano "${TOOL_SERVICE}"



echo Activate ${TOOL_NAME} service
sudo systemctl daemon-reload
sudo systemctl enable "${TOOL_NAME}.service"
sudo systemctl restart "${TOOL_NAME}.service"
sleep 3
sudo systemctl status "${TOOL_NAME}.service"



echo Enable access to ${TOOL_PORT}
sudo ufw allow to 0.0.0.0/0 port ${TOOL_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_WEB_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo Check ${TOOL_NAME} service
curl "http://localhost:${TOOL_WEB_PORT}/gui/
sudo lsof -i -P -n | grep "${TOOL_WEB_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"

echo "Login to http://localhost:${TOOL_WEB_PORT}/gui/"
echo "Set credentials: ${TOOL_NAME}_user ${USER_PASSWORD}"
```
