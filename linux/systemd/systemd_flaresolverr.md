# Enable `ntp`-based synchronization

See `linux_ntp.md` for details

# Configure FlareSolverr

```shell script
echo Export variables
#
export TOOL_NAME="FlareSolverr"
export TOOL_PORT=8191
export IMG="ghcr.io/flaresolverr/flaresolverr:latest"
#
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"

# Do not create the separate user
# The default user's UID/GID 1000/1000 are the Docker image internal UID/GID (check below)
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
    "${TOOL_DIR}"
sudo chown \
    --recursive \
    --verbose \
    "1000:1000" \
    "${TOOL_DIR}"
```

## Inspect Docker image

```shell script
docker pull "${IMG}"
docker run \
    --env LOG_LEVEL="${LOG_LEVEL}" \
    --env PORT="${TOOL_PORT}" \
    --entrypoint /bin/sh \
    --interactive \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    "${IMG}"

id -u  # 1000
id -g  # 1000
/usr/local/bin/python -u /app/flaresolverr.py
```

## Configure and start tool

```shell script
echo Create tool script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_PORT="${TOOL_PORT}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "LOG_LEVEL=${LOG_LEVEL}" \\
    --env "PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}" \\
    --rm \\
    --user 1000 \\
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
User=1000
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
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep ${TOOL_NAME}
```
