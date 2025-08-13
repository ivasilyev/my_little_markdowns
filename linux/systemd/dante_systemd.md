# Deploy tool

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="dante"
export USER_NAME="${TOOL_NAME}_user"
export USER_PASSWORD=""
export TOOL_PORT=
export TOOL_DIR="/opt/${TOOL_NAME}/"
export TOOL_DATA_DIR="/var/opt/${TOOL_NAME}/"
export TOOL_CFG="${TOOL_DIR}${TOOL_NAME}.conf"
export TOOL_PASSWD_FILE="${TOOL_DATA_DIR}passwd"
export TOOL_SHADOW_FILE="${TOOL_DATA_DIR}shadow"
export TOOL_SCRIPT="${TOOL_DIR}${TOOL_NAME}.sh"
export TOOL_SERVICE="/etc/systemd/system/${TOOL_NAME}.service"
export IMG="ghcr.io/aeron/socks5-dante-proxy:24.1"

echo Create user "${USER_NAME}"
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
sudo touch \
    "${TOOL_CFG}" \
    "${TOOL_PASSWD_FILE}" \
    "${TOOL_SHADOW_FILE}"
sudo chown \
    --recursive \
    --verbose \
    "root" \
    "${TOOL_DIR}" \
    "${TOOL_DATA_DIR}"
```

## Pre-configure tool

```shell script
docker pull "${IMG}"
docker run \
    --entrypoint /bin/sh \
    --env USER_NAME="${USER_NAME}" \
    --env USER_PASSWORD="${USER_PASSWORD}" \
    --interactive \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user 0 \
    --volume "${TOOL_CFG}:/etc/danted.conf" \
    --volume "${TOOL_PASSWD_FILE}:/etc/passwd1" \
    --volume "${TOOL_SHADOW_FILE}:/etc/shadow1" \
    "${IMG}"

# passwd manipulates a temporary file, and then attempts to rename it to /etc/shadow
# it fails because /etc/shadow is a mountpoint - which cannot be replaced

ls -la /etc
# Does not work
# sh /srv/entrypoint.sh add-user "${USER_NAME}" "${USER_PASSWORD}" &
userdel "${USER_NAME}"
useradd \
    --comment '${TOOL_NAME} service user' \
    --shell "/usr/bin/false" \
    --no-create-home \
    --no-user-group \
    --system \
    "${USER_NAME}"
echo "${USER_NAME}:${USER_PASSWORD}" | chpasswd

sleep 3

cat /etc/passwd > /etc/passwd1
cat /etc/shadow > /etc/shadow1

exit

docker run \
    --entrypoint /bin/sh \
    --interactive \
    --env NPROC="$(nproc)" \
    --env TOOL_CFG="${TOOL_CFG}" \
    --name "${TOOL_NAME}" \
    --publish "${TOOL_PORT}:${TOOL_PORT}" \
    --rm \
    --tty \
    --user 0 \
    --volume "${TOOL_CFG}:${TOOL_CFG}" \
    --volume "${TOOL_PASSWD_FILE}:/etc/passwd" \
    --volume "${TOOL_SHADOW_FILE}:/etc/shadow" \
    "${IMG}"

# sh /srv/entrypoint.sh start &
danted -N "${NPROC}" -f "${TOOL_CFG}"
```

## Configure and start tool

```shell script
echo Create ${TOOL_NAME} configuration file
cat <<EOF | sudo tee "${TOOL_CFG}"
logoutput: syslog 
errorlog: stdout 
user.privileged: root
user.unprivileged: nobody

# The listening network interface or address.
internal: 0.0.0.0 port = ${TOOL_PORT}

# The proxying network interface or address.
# external: $(ip route | grep 'default' | grep --only-matching --perl-regex '(?<= dev )[^ ]+')
# external: eth0
external: 127.0.0.1

# client-rules determine who can connect to the internal interface.
clientmethod: none

# socks-rules determine what is proxied through the external interface.
# enable for protection
# socksmethod: username
socksmethod: none

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: error connect disconnect
}

client block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: bind connect udpassociate
    log: error connect disconnect
    # enable for protection
    # socksmethod: username
    socksmethod: none
}

socks block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}
EOF
# nano "${TOOL_CFG}"

echo Create ${TOOL_NAME} routine script
cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/usr/bin/env bash
# bash "${TOOL_SCRIPT}"
export TOOL_NAME="${TOOL_NAME}"
export TOOL_CFG="${TOOL_CFG}"
export TOOL_PASSWD_FILE="${TOOL_PASSWD_FILE}"
export TOOL_SHADOW_FILE="${TOOL_SHADOW_FILE}"
export TOOL_PORT="${TOOL_PORT}"
export NPROC="\$(nproc)"

export IMG="${IMG}"
docker pull "\${IMG}"
docker run \\
    --entrypoint /usr/sbin/danted \\
    --name "\${TOOL_NAME}" \\
    --net host \\
    --rm \\
    --volume "\${TOOL_CFG}:\${TOOL_CFG}" \\
    --volume "\${TOOL_PASSWD_FILE}:/etc/passwd" \\
    --volume "\${TOOL_SHADOW_FILE}:/etc/shadow" \\
    --user "root" \\
    "\${IMG}" \\
        -N "\${NPROC}" \\
        -f "\${TOOL_CFG}"
EOF

sudo chmod a+x -v "${TOOL_SCRIPT}"
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

# curl -v -x "socks5://localhost:${TOOL_PORT}" http://ifconfig.co
curl -v -x "socks5://${USER_NAME}:${USER_PASSWORD}@localhost:${TOOL_PORT}" http://ifconfig.co
curl --socks5 ${USER_NAME}:${USER_PASSWORD}@localhost:${TOOL_PORT} -L http://ifconfig.co
curl --socks5 ${USER_NAME}:${USER_PASSWORD}@localhost:${TOOL_PORT} -L http://ifconfig.co)
sudo lsof -i -P -n | grep "${TOOL_PORT}"
pgrep "${TOOL_NAME}"
```
