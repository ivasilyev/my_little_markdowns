# Deploy Suwayomi Server

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="suwayomi_server"
export USER_NAME="$(whoami)"
export TOOL_PORT=4567
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_DATA_DIR="/var/opt/${TOOL_NAME}/"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export IMG="ghcr.io/suwayomi/tachidesk:v2.2.2141"

echo Create directories
sudo rm \
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo mkdir \
    --parent \
    --mode 0700 \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
```

## Inspect & debug Docker image if it does contain shell

```shell script
docker pull "${IMG}"
# Note the mount order
# The order matters! Make sure the downloads is first in the volume list or it will not work!
docker run \
    --entrypoint /bin/sh \
    --interactive \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --volume "${TOOL_DATA_DIR}:/home/suwayomi/.local/share/Tachidesk/downloads" \
    --volume "${TOOL_DIR}:/home/suwayomi/.local/share/Tachidesk" \
    "${IMG}"

echo "$(id -u) / $(id -g)"  # 1000 / 1000 makes the different user less applicable

bash /home/suwayomi/startup_script.sh
```

## Configure and start tool

```shell script
echo Create ${TOOL_NAME} routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DIR="${TOOL_DIR}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_PORT="${TOOL_PORT}"
export USER_NAME="${USER_NAME}"

export IMG="${IMG}"
docker pull "\${IMG}"
# The order matters! Make sure the downloads is first in the volume list or it will not work!
docker run \\
    --name "\${TOOL_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/udp" \\
    --rm \\
    --volume "\${TOOL_DATA_DIR}:/home/suwayomi/.local/share/Tachidesk/downloads" \\
    --volume "\${TOOL_DIR}:/home/suwayomi/.local/share/Tachidesk" \\
    --user "\$(id --user "\${USER_NAME}")" \\
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



echo Check ${TOOL_NAME} service
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"
```
