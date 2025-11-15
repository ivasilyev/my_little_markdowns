# Deploy Alertmanager

## Prepare environment

```shell script
echo Export variables
#
export TOOL_NAME="alertmanager"
export TOOL_DATA_DIR="/data/${TOOL_NAME}"
export TOOL_PORT=9093
export TOOL_CLUSTERING_PORT=9094
export USER_PASSWORD=""
export IMG="prom/alertmanager:latest"
#
export USER_NAME="${TOOL_NAME}-user"
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_CFG="${TOOL_DIR}${TOOL_NAME}.conf"
export TOOL_WEB_CFG="${TOOL_DIR}${TOOL_NAME}-web.conf"
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
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}"
sudo mkdir \
    --mode 0700 \
    --parent \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo touch "${TOOL_CFG}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"

echo Create network
docker network create --driver=bridge "${TOOL_NETWORK}"

echo Install htpasswd
sudo apt-get install -y apache2-utils
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

/bin/alertmanager -h
```

## Configure and start tool

```shell script
echo Create tool configuration file
cat <<EOF | sudo tee "${TOOL_CFG}"
---
global:
  resolve_timeout: 5m
route:
  receiver: main
  group_by:
  - job
  routes:
  - receiver: "null"
    match:
      alertname: Watchdog
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
receivers:
- name: "null"
- name: "main"
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal:
    - "alertname"
    - "dev"
    - "instance"
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



echo Create tool routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export NETWORK_NAME="${NETWORK_NAME}"
export TOOL_CFG="${TOOL_CFG}"
export TOOL_WEB_CFG="${TOOL_WEB_CFG}"
export TOOL_CLUSTERING_PORT="${TOOL_CLUSTERING_PORT}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_DIR="${TOOL_DIR}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_PORT="${TOOL_PORT}"
export USER_NAME="${USER_NAME}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "TOOL_CFG=\${TOOL_CFG}" \\
    --env "TOOL_DATA_DIR=\${TOOL_DATA_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --network "\${NETWORK_NAME}" \\
    --publish "\${TOOL_CLUSTERING_PORT}:\${TOOL_CLUSTERING_PORT}/tcp" \\
    --publish "\${TOOL_CLUSTERING_PORT}:\${TOOL_CLUSTERING_PORT}/udp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/udp" \\
    --rm \\
    --user "\$(id --user "\${USER_NAME}")" \\
    --volume "\${TOOL_DIR}:\${TOOL_DIR}" \\
    --volume "\${TOOL_DATA_DIR}:\${TOOL_DATA_DIR}" \\
    "\${IMG}" \\
        --log.level=error \\
        --config.file="\${TOOL_CFG}" \\
        --storage.path="\${TOOL_DATA_DIR}" \\
        --web.config.file="\${TOOL_WEB_CFG}" \\
        --web.listen-address=:\${TOOL_PORT}
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



echo Enable access to ${TOOL_CLUSTERING_PORT}
sudo ufw allow proto tcp to 0.0.0.0/0 port ${TOOL_CLUSTERING_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw allow proto udp to 0.0.0.0/0 port ${TOOL_CLUSTERING_PORT} comment "${TOOL_NAME} server listen port"
sudo ufw --force disable
sudo ufw --force enable
sudo ufw status verbose



echo Check ${TOOL_NAME} service
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"
```
