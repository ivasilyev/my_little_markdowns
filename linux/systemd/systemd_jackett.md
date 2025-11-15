# Configure Jackett

```shell script
echo Export variables
#
export TOOL_NAME="jackett"
export USER_PASSWORD=""
export TOOL_PORT=9117
export TOOL_DATA_DIR="/data/${TOOL_NAME}"
export IMG="linuxserver/jackett:latest"
#
export USER_NAME="${TOOL_NAME}-user"
export TOOL_DIR="/opt/${TOOL_NAME}/"
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
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}"
sudo mkdir \
    --parent \
    --verbose \
    --mode 0700 \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}"):$(id --group "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
```

## Configure and start tool

```shell script
echo Create tool script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export USER_NAME="${USER_NAME}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_CFG="${TOOL_CFG}"
export TOOL_PORT="${TOOL_PORT}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "TOOL_DATA_DIR=\${TOOL_DATA_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --publish "\${TOOL_PORT}:9117/tcp" \\
    --rm \\
    --user "\$(id --user "\${USER_NAME}")" \\
    --volume "\${TOOL_DATA_DIR}:/config" \\
    "\${IMG}"
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

echo "Set Admin password at the Web UI: http://${TOOL_NAME}:${TOOL_PORT}/UI/Dashboard"
```
