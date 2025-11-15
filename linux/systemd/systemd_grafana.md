# Deploy Grafana

## Prepare environment

```shell script
echo Export variables
#
export TOOL_NAME="grafana"
export TOOL_PORT=3000
export TOOL_DATA_DIR="/data/${TOOL_NAME}/"
export USER_PASSWORD=""
export IMG="grafana/grafana-oss:latest"
#
export USER_NAME="${TOOL_NAME}-user"
export TOOL_DIR="/opt/${TOOL_NAME}/"
# Avoid Error: x migration failed (id = create migration_log table): database is locked
export TOOL_DATA_DIR="/${TOOL_NAME}/data/"
# Must be an INI file
export TOOL_CFG="${TOOL_DIR}${TOOL_NAME}.ini"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export TOOL_NETWORK="monitoring"
# The port seems to be actually hardcoded inside the program
export TOOL_RENDERER_PORT=8081

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
# UID 472 is the Docker image internal UID (check below)
sudo usermod \
    --uid 472 \
    --gid 0 \
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
    "${TOOL_DIR}provisioning/alerting" \
    "${TOOL_DIR}provisioning/dashboards" \
    "${TOOL_DIR}provisioning/datasources" \
    "${TOOL_DIR}provisioning/notifiers" \
    "${TOOL_DIR}provisioning/plugins" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}" \
    "${TOOL_DATA_DIR}plugins"
sudo touch "${TOOL_CFG}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}"):$(id --group "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"

echo Create network
docker network create --driver=bridge "${TOOL_NETWORK}"
```

## Inspect Docker image

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env TOOL_CFG="${TOOL_CFG}" \
    --env TOOL_DATA_DIR="${TOOL_DATA_DIR}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --network "${TOOL_NETWORK}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --volume "${TOOL_DIR}:${TOOL_DIR}" \
    --volume "${TOOL_DATA_DIR}:${TOOL_DATA_DIR}" \
    "${IMG}"

/usr/share/grafana/bin/grafana -h
id
id grafana  # 472
echo test > "${TOOL_DATA_DIR}/test"
cat "${TOOL_DATA_DIR}/test"
```

## Configure and start tool

```shell script
echo "Create tool configuration file"
cat <<EOF | sudo tee "${TOOL_CFG}"
[server]
protocol = http
http_addr = 0.0.0.0
http_port = ${TOOL_PORT}
root_url = http://0.0.0.0:${TOOL_PORT}

[paths]
# Do not work here - use mocking mounts instead
# data = "${TOOL_DATA_DIR}"
# plugins = "${TOOL_DATA_DIR}plugins"

[database]
type = sqlite3
host = 127.0.0.1:3306
name = grafana
user = root
password =
url =
max_idle_conn = 2
max_open_conn =
conn_max_lifetime = 14400
log_queries = false
ssl_mode = disable
isolation_level =
ca_cert_path =
client_key_path =
client_cert_path =
server_cert_name =
path = grafana.db
cache_mode = private
locking_attempt_timeout_sec = 0

[users]
viewers_can_edit = false

[auth]
disable_login_form = false
disable_signout_menu = false

[auth.anonymous]
enabled = false

[auth.basic]
enabled = true

[log]
mode = console
level = error

[metrics]
enabled = true
interval_seconds  = 10
disable_total_stats = false
total_stats_collector_interval_seconds = 1800

[dashboards.json]
enabled = true
path = "${TOOL_DATA_DIR}dashboards"

[rendering]
server_url = "http://grafana-image-renderer:${TOOL_RENDERER_PORT}/render"
callback_url = "http://grafana:${TOOL_PORT}/"

[security]
admin_user = ${USER_NAME}
admin_password = ${USER_PASSWORD}
EOF

# nano "${TOOL_CFG}"

echo "Create tool routine script"
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DATA_DIR="${TOOL_DATA_DIR}"
export TOOL_DIR="${TOOL_DIR}"
export TOOL_PORT="${TOOL_PORT}"
export TOOL_RENDERER_PORT="${TOOL_RENDERER_PORT}"
export USER_NAME="${USER_NAME}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "GF_PLUGINS_PREINSTALL=grafana-clock-panel,grafana-image-renderer" \\
    --env "TOOL_DATA_DIR=\${TOOL_DATA_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --network "${TOOL_NETWORK}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}" \\
    --rm \\
    --volume "\${TOOL_DIR}:/etc/grafana/" \\
    --volume "\${TOOL_DATA_DIR}:/var/lib/grafana" \\
    --user "\$(id --user "${USER_NAME}")" \\
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
curl "http://localhost:${TOOL_PORT}"
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"



# Fix Error: migration failed (id = create migration_log table): database is locked
sudo systemctl stop "${TOOL_NAME}.service"
sleep 5
sudo rm -rfv "${TOOL_DATA_DIR}"/*
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
sudo systemctl restart "${TOOL_NAME}.service"
sleep 5
sudo systemctl status "${TOOL_NAME}.service"
sudo journalctl -xe
```
