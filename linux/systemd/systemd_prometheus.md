# Enable `ntp`-based synchronization

See `linux_ntp.md` for details

# Configure Prometheus

```shell script
echo Export variables
#
export TOOL_NAME="prometheus"
export USER_PASSWORD=""
export TOOL_PORT=9090
export TOOL_DATA_DIR="/data/${TOOL_NAME}"
export NODE_EXPORTER_PORT=9100
export IMG="prom/prometheus:latest"
#
export USER_NAME="${TOOL_NAME}-user"
export NETWORK_NAME="monitoring"
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_CFG="${TOOL_DIR}${TOOL_NAME}.conf"
export TOOL_WEB_CFG="${TOOL_DIR}${TOOL_NAME}-web.conf"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
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
    -rf \
    "${TOOL_DIR}"
sudo mkdir \
    --parent \
    --mode 0700 \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}"):$(id --group "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"

echo Install htpasswd
sudo apt-get install -y apache2-utils
```

## Configure and start tool

```shell script
echo Create tool configuration file
# export TOOL_CFG="${TOOL_CFG}" && rm -f "\${TOOL_CFG}" && sudo nano "${TOOL_CFG}"
cat <<EOF | sudo tee "${TOOL_CFG}"
---
global:
  scrape_interval: 15s
  evaluation_interval: 15s
EOF

# nano "${TOOL_CFG}"



echo Create tool web configuration file
cat <<EOF | sudo tee "${TOOL_WEB_CFG}"
# export TOOL_WEB_CFG="${TOOL_WEB_CFG}" && rm -f "\${TOOL_WEB_CFG}" && sudo nano "${TOOL_WEB_CFG}"
---
basic_auth_users:
    ${USER_NAME}: $(htpasswd -B -C 10 -n -b "${USER_NAME}" "${USER_PASSWORD}" | cut -d ":" -f 2)
EOF

# nano "${TOOL_WEB_CFG}"



echo Create tool script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export USER_NAME="${USER_NAME}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_CFG="${TOOL_CFG}"
export TOOL_WEB_CFG="${TOOL_WEB_CFG}"
export TOOL_PORT="${TOOL_PORT}"
export NETWORK_NAME="${NETWORK_NAME}"

export IMG="${IMG}"
docker network create "\${NETWORK_NAME}"
docker pull "\${IMG}"
docker run \\
    --env "TOOL_CFG=\${TOOL_CFG}" \\
    --env "TOOL_WEB_CFG=\${TOOL_WEB_CFG}" \\
    --env "TOOL_DATA_DIR=\${TOOL_DATA_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --network "\${NETWORK_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}" \\
    --rm \\
    --user "\$(id --user "\${USER_NAME}")" \\
    --volume "\${TOOL_CFG}:\${TOOL_CFG}" \\
    --volume "\${TOOL_WEB_CFG}:\${TOOL_WEB_CFG}" \\
    --volume "\${TOOL_DATA_DIR}:\${TOOL_DATA_DIR}" \\
    "\${IMG}" \\
        --config.file="\${TOOL_CFG}" \\
        --web.config.file="\${TOOL_WEB_CFG}" \\
        --log.level=error \\
        --storage.tsdb.path="\${TOOL_DATA_DIR}" \\
        --storage.tsdb.retention.time="1h" \\
        --web.enable-lifecycle \\
        --web.listen-address="0.0.0.0:\${TOOL_PORT}"
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
sudo systemctl status "${TOOL_NAME}.service"



echo Enable access to ${TOOL_PORT}
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo "Check ${TOOL_NAME} service"
sleep 5
curl "http://localhost:${TOOL_PORT}/metrics"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep ${TOOL_NAME}
```
