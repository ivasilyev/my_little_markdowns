# Deploy Grafana image renderer

## Prepare environment

```shell script
# Run after Grafana setup
echo Export variables
#
export TOOL_NAME="grafana-image-renderer"
export IMG="grafana/grafana-image-renderer:latest"
#
# export USER_PASSWORD=""
export USER_NAME="root"
export TOOL_DIR="/opt/grafana/"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export TOOL_NETWORK="monitoring"
# The port seems to be actually hardcoded inside the program
export TOOL_PORT=8081
```

## Inspect Docker image

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --interactive \
    --name "${TOOL_NAME}" \
    --network "${TOOL_NETWORK}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    "${IMG}"


/usr/bin/grafana-image-renderer --help

id -u  # 65532
id -g  # 996
```

## Configure and start tool

```shell script
echo "Create tool routine script"
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_PORT="${TOOL_PORT}"

export IMG="${IMG}"
docker pull "\${IMG}"
# The HTTP_PORT is hardcoded
docker run \\
    --env "ENABLE_METRICS=true" \\
    --env "HTTP_HOST=0.0.0.0" \\
    --env "HTTP_PORT=\${TOOL_PORT}" \\
    --env "HTTP_PROTOCOL=http" \\
    --env "IGNORE_HTTPS_ERRORS=true" \\
    --env "LOG_LEVEL=error" \\
    --env "RENDERING_ARGS=--no-sandbox,--disable-setuid-sandbox,--disable-dev-shm-usage,--disable-accelerated-2d-canvas,--disable-gpu,--window-size=1280x758" \\
    --env "RENDERING_CLUSTERING_MAX_CONCURRENCY=9" \\
    --env "RENDERING_CLUSTERING_MODE=browser" \\
    --env "RENDERING_CLUSTERING_TIMEOUT=30" \\
    --env "RENDERING_MODE=clustered" \\
    --env "RENDERING_VERBOSE_LOGGING=false" \\
    --name "${TOOL_NAME}" \\
    --network "${TOOL_NETWORK}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --rm \\
    --user "$(id --user "${USER_NAME}")" \\
    "\${IMG}"
EOF

# nano "${TOOL_SCRIPT}"
sudo chmod a+x "${TOOL_SCRIPT}"



echo "Create system service"
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



echo "Activate ${TOOL_NAME} service"
sudo systemctl daemon-reload
sudo systemctl enable "${TOOL_NAME}.service"
sudo systemctl restart "${TOOL_NAME}.service"
sleep 3
sudo systemctl status "${TOOL_NAME}.service"



echo "Enable access to ${TOOL_PORT}"
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo "Check ${TOOL_NAME} service"
curl "http://localhost:${TOOL_PORT}/render/version"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
ps aux | grep "${TOOL_NAME}"
```
