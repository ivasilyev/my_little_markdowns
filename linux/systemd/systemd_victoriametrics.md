# Deploy VictoriaMetrics

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="victoriametrics"
export USER_PASSWORD=""
export TOOL_PORT=8428
export TOOL_DATA_DIR="/data/${TOOL_NAME}"
export IMG="victoriametrics/victoria-metrics:latest"

export USER_NAME="${TOOL_NAME}-user"
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_CFG="${TOOL_DIR}${TOOL_NAME}.conf"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export NETWORK_NAME="monitoring"

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
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo mkdir \
    --parent \
    --mode 0700 \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo chown \
    --recursive \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
```

## Inspect Docker image

```shell script
docker network create "${NETWORK_NAME}"
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env TOOL_CFG="${TOOL_CFG}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --network "${NETWORK_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user "$(id --user "${USER_NAME}")" \
    --volume "${TOOL_DATA_DIR}:${TOOL_DATA_DIR}" \
    "${IMG}"

/victoria-metrics-prod -h


docker network create "${NETWORK_NAME}"
export IMG="victoriametrics/vmauth:latest"
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env TOOL_CFG="${TOOL_CFG}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --network "${NETWORK_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user "$(id --user "${USER_NAME}")" \
    --volume "${TOOL_DATA_DIR}:${TOOL_DATA_DIR}" \
    "${IMG}"
```

## Configure and start tool

```shell script
echo Create tool script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_CFG="${TOOL_CFG}"
export TOOL_PORT="${TOOL_PORT}"
export USER_NAME="${USER_NAME}"
export NETWORK_NAME="${NETWORK_NAME}"

export IMG="${IMG}"
docker network create "\${NETWORK_NAME}"
docker pull "\${IMG}"
docker run \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --env "TOOL_DATA_DIR=\${TOOL_DATA_DIR}" \\
    --name "\${TOOL_NAME}" \\
    --network "\${NETWORK_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}" \\
    --rm \\
    --volume "\${TOOL_DATA_DIR}:\${TOOL_DATA_DIR}" \\
    --user "$(id --user "${USER_NAME}")" \\
    "\${IMG}" \\
        -httpListenAddr "0.0.0.0:\${TOOL_PORT}" \\
        -httpAuth.username "${USER_NAME}" \\
        -httpAuth.password "${USER_PASSWORD}" \\
        -loggerLevel ERROR \\
        -retentionPeriod 3y \\
        -storageDataPath "\${TOOL_DATA_DIR}"      
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



echo Check ${TOOL_NAME} service
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep victoria-metric
```

# Configure Prometheus

```shell script
cat <<EOF | sudo tee -a "/opt/prometheus/prometheus.conf"
remote_write:
  - url: http://${TOOL_NAME}:${TOOL_PORT}/api/v1/write
    queue_config:
      max_samples_per_send: 10000
      capacity: 20000
      max_shards: 30
EOF
# nano "/opt/prometheus/prometheus.conf"

sudo systemctl restart prometheus.service
sudo systemctl status prometheus.service
```
