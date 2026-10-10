# Deploy lampac

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="lampac"
export USER_PASSWORD=""
export TOOL_PORT=9118
export IMG="ghcr.io/lampac-nextgen/lampac:v1.73.0"
#
# lampac user already has uid 1000
export USER_NAME="$(whoami)"
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_DATA_DIR="/data/.${TOOL_NAME}/"
export TOOL_CFG_FILE="${TOOL_DIR}init.conf"
export TOOL_PASSWD_FILE="${TOOL_DIR}passwd"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export TOOL_NETWORK="${TOOL_NAME}"

echo Create directories
sudo rm \
    --force \
    --recursive \
    --verbose \
    "${TOOL_DIR}"
sudo mkdir \
    --parent \
    --mode 0777 \
    --verbose \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}cache" \
    "${TOOL_DIR}database" \
    "${TOOL_DIR}mods" \
    "${TOOL_DIR}plugins"

sudo touch "${TOOL_CFG_FILE}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}")" \
    "${TOOL_DIR}"
sudo chmod -Rv 0777 "${TOOL_DIR}"


echo Create network
docker network create \
    --driver=bridge \
    --subnet=10.10.10.0/24 \
    "${TOOL_NETWORK}"
```

## Inspect & debug Docker image if it does contain shell

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env TOOL_CFG_FILE="${TOOL_CFG_FILE}" \
    --interactive \
    --ip 10.10.10.10 \
    --name "${TOOL_NAME}" \
    --network "${TOOL_NETWORK}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user "$(id --user "${USER_NAME}")" \
    --volume "${TOOL_DIR}:${TOOL_DIR}" \
    --volume "${TOOL_DATA_DIR}:${TOOL_DATA_DIR}" \
    "${IMG}"

"/usr/share/dotnet/dotnet" "/lampac/Core.dll"
```

## Configure and start tool

```shell script
echo Create ${TOOL_NAME} configuration files
cat <<EOF | tee "${TOOL_CFG_FILE}"
{
  "accsdb": {
    "enable": true,
    "users": [
      { "id": "user", "expires": "2199-12-31" }
    ]
  },
  "listen": {
    "version": true,
    "ip": "0.0.0.0",
    "port": ${TOOL_PORT},
    "scheme": "http",
    "localhost": "127.0.0.1"
  },
  "BaseModule": {
    "SkipModules": [
      "Catalog",
      "DLNA",
      "Sync",
      "TimeCode",
      "Tracks",
      "Transcoding",
      "WebLog"
    ]
  },
  "chromium": {
    "enable": true,
    "executablePath": "/usr/bin/chromium"
  },
  "openstat": {
    "enable": true
  },
  "online": {
    "name": "Lampac NextGen",
    "spiderName": "Lampac NextGen",
    "version": true,
    "btn_priority_forced": true
  },
  "sisi": {
    "lgbt": false,
    "spider": false,
    "NextHUB": true,
    "history": {
      "enable": false
    }
  }
}
EOF
# nano "${TOOL_CFG_FILE}"

echo "${USER_PASSWORD}" | tee "${TOOL_PASSWD_FILE}"
# nano "${TOOL_PASSWD_FILE}"

cat <<EOF | tee "${TOOL_DIR}users.json"
[
  {
    "id": 1,
    "uid": "password",
    "name": "LocalUser",
    "role": "user",
    "account": {
      "expired": "2199-12-31"
    },
    "accs": {
      "group": "user"
    }
  }
]
EOF

cat <<EOF | tee "${TOOL_DIR}admin_panel_manifest.json"
{
  "enable": true,
  "dynamic": true,
  "tree": [
    "AdminPanelController.cs",
    "ConfigSectionGroups.cs",
    "ModInit.cs"
  ]
}
EOF

echo Create ${TOOL_NAME} routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_DIR="${TOOL_DIR}"
export TOOL_CFG_FILE="${TOOL_CFG_FILE}"
export TOOL_PASSWD_FILE="${TOOL_PASSWD_FILE}"
export TOOL_PORT="${TOOL_PORT}"
export USER_NAME="${USER_NAME}"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --env "TOOL_CFG_FILE=\${TOOL_CFG_FILE}" \\
    --env "TOOL_DIR=\${TOOL_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/udp" \\
    --rm \\
    --volume "\${TOOL_CFG_FILE}:/lampac/init.conf" \\
    --volume "\${TOOL_DIR}users.json:/lampac/users.json" \\
    --volume "\${TOOL_PASSWD_FILE}:/lampac/passwd" \\
    --volume "\${TOOL_DIR}admin_panel_manifest.json:/lampac/module/AdminPanel/manifest.json" \\
    --volume "\${TOOL_DIR}cache:/lampac/cache" \\
    --volume "\${TOOL_DIR}database:/lampac/database" \\
    --volume "\${TOOL_DIR}mods:/lampac/mods" \\
    --volume "\${TOOL_DIR}plugins:/lampac/plugins" \\
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
curl "http://localhost:${TOOL_PORT}/version?type=hash"
curl "http://localhost:${TOOL_PORT}/online.js"
curl "http://localhost:${TOOL_PORT}/adminpanel/auth"
```
