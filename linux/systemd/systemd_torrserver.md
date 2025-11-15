# Deploy TorrServer

## Prepare environment

```shell script
echo Export variables
#
export TOOL_NAME="torrserver"
export TOOL_DATA_DIR="/data/${TOOL_NAME}/"
export TOOL_PORT=8090
export TOOL_PEERS_LISTEN_PORT=49152
export USER_PASSWORD=""
export IMG="ghcr.io/yourok/torrserver:latest"
#
export USER_NAME="${TOOL_NAME}-user"
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_CFG="${TOOL_DIR}settings.json"
export TOOL_AUTH_CFG="${TOOL_DIR}accs.db"
export TOOL_TORRENT_DIR="${TOOL_DATA_DIR}torrents"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"


echo Create user "${USER_NAME}"
sudo userdel "${USER_NAME}"
sudo useradd \
    --comment '${TOOL_NAME} service user' \
    --shell "/usr/bin/false" \
    --no-create-home \
    --no-user-group \
    --system \
    "${USER_NAME}"
echo "${USER_NAME}:${USER_PASSWORD}" | sudo chpasswd
sudo usermod \
    --append \
    --groups \
    docker \
    "${USER_NAME}"

echo Create directories
sudo rm \
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}"
sudo mkdir \
    --mode 0700 \
    --parent \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}" \
    "${TOOL_TORRENT_DIR}"
sudo touch "${TOOL_CFG}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}" \
    "${TOOL_TORRENT_DIR}"
```

## Inspect Docker image

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env TOOL_CFG="${TOOL_CFG}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user "$(id --user "${USER_NAME}")" \
    --volume "${TOOL_DIR}:${TOOL_DIR}" \
    --volume "${TOOL_DATA_DIR}:${TOOL_DATA_DIR}" \
    "${IMG}"

/usr/bin/torrserver -h
```

## Configure and start tool

```shell script
echo Create tool configuration file
cat <<EOF | sudo tee "${TOOL_CFG}"
{
  "BitTorr": {
    "CacheSize": 4294967296,
    "ConnectionsLimit": 30,
    "DisableDHT": false,
    "DisablePEX": false,
    "DisableTCP": false,
    "DisableUPNP": false,
    "DisableUTP": false,
    "DisableUpload": false,
    "DownloadRateLimit": 0,
    "EnableDLNA": false,
    "EnableDebug": false,
    "EnableIPv6": false,
    "EnableRutorSearch": true,
    "ForceEncrypt": true,
    "FriendlyName": "",
    "PeersListenPort": ${TOOL_PEERS_LISTEN_PORT},
    "PreloadCache": 0,
    "ReaderReadAHead": 95,
    "RemoveCacheOnDrop": false,
    "ResponsiveMode": false,
    "RetrackersMode": 1,
    "SslCert": "",
    "SslKey": "",
    "SslPort": 0,
    "TorrentDisconnectTimeout": 30,
    "TorrentsSavePath": "",
    "UploadRateLimit": 0,
    "UseDisk": false
  }
}
EOF

# nano "${TOOL_CFG}"



echo Create tool authentication configuration file
cat <<EOF | sudo tee "${TOOL_AUTH_CFG}"
{
    "${USER_NAME}": "${USER_PASSWORD}"
}
EOF

# nano "${TOOL_AUTH_CFG}"



echo Create tool routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_DIR="${TOOL_DIR}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_TORRENT_DIR="${TOOL_TORRENT_DIR}"
export TOOL_PEERS_LISTEN_PORT=${TOOL_PEERS_LISTEN_PORT}
export TOOL_PORT=${TOOL_PORT}
export USER_NAME="${USER_NAME}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --entrypoint /usr/bin/torrserver \\
    --env "TOOL_DIR=\${TOOL_DIR}" \\
    --env "TOOL_TORRENT_DIR=\${TOOL_TORRENT_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "${TOOL_NAME}" \\
    --publish "\${TOOL_PEERS_LISTEN_PORT}:\${TOOL_PEERS_LISTEN_PORT}/tcp" \\
    --publish "\${TOOL_PEERS_LISTEN_PORT}:\${TOOL_PEERS_LISTEN_PORT}/udp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/udp" \\
    --rm \\
    --user "\$(id --user "\${USER_NAME}")" \\
    --volume "\${TOOL_DIR}:\${TOOL_DIR}" \\
    --volume "\${TOOL_DATA_DIR}:\${TOOL_DATA_DIR}" \\
    "\${IMG}" \\
        --httpauth \\
        --path="\${TOOL_DIR}" \\
        --port=\${TOOL_PORT} \\
        --torrentsdir="\${TOOL_TORRENT_DIR}"
EOF

sudo chmod a+x "${TOOL_SCRIPT}"
# nano "${TOOL_SCRIPT}"



echo Create system service
cat <<EOF | sudo tee "${TOOL_SERVICE}"
[Unit]
Description=${TOOL_NAME}
Documentation=https://google.com
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=${USER_NAME}
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
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw allow proto udp to 0.0.0.0/0 port ${TOOL_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo Enable access to ${TOOL_PEERS_LISTEN_PORT}
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_PEERS_LISTEN_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw allow proto udp to 0.0.0.0/0 port ${TOOL_PEERS_LISTEN_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo Check ${TOOL_NAME} service
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"
```
