# Deploy lampac

## Prepare environment

```shell script
echo Export variables
export TOOL_NAME="lampac"
export USER_PASSWORD=""
export TOOL_PORT=9118
export IMG="ghcr.io/lampac-nextgen/lampac:v1.82.0"
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
    "${TOOL_DIR}data" \
    "${TOOL_DIR}database" \
    "${TOOL_DIR}mods" \
    "${TOOL_DIR}plugins"

sudo touch "${TOOL_CFG_FILE}"
sudo chown \
    --recursive \
    --verbose \
    "$(id --user "${USER_NAME}"):$(id --group "${USER_NAME}")" \
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
    --entrypoint /bin/bash \
    --env TOOL_CFG_FILE="${TOOL_CFG_FILE}" \
    --interactive \
    --network "${TOOL_NETWORK}" \
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
echo "${USER_PASSWORD}" | tee "${TOOL_PASSWD_FILE}"
# nano "${TOOL_PASSWD_FILE}"

echo "Get plugins"
curl "http://cub.red/plugin/tmdb-proxy" \
    -o "${TOOL_DIR}mods/cub-tmdb-proxy.js"
curl "http://cub.red/plugin/tracks" \
    -o "${TOOL_DIR}mods/cub-tracks.js"
curl "https://plugin.rootu.top/tmdb.js" \
    -o "${TOOL_DIR}mods/rootu-tmdb-proxy.js"
curl "https://bylampa.github.io/tmdb-proxy.js" \
    -o "${TOOL_DIR}mods/bylampa-tmdb-proxy.js"
# https://github.com/YouROK/TorrServer/releases?page=2#release-MatriX.135
curl "https://release-assets.githubusercontent.com/github-production-release-asset/146576102/05a2e954-e163-445c-b45f-36d9cb02ef98?sp=r&sv=2018-11-09&sr=b&spr=https&se=2026-10-14T19%3A52%3A54Z&rscd=attachment%3B+filename%3DTorrServer-linux-amd64&rsct=application%2Foctet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2026-10-14T18%3A52%3A32Z&ske=2026-10-14T19%3A52%3A54Z&sks=b&skv=2018-11-09&sig=v0mc5DjTqfsz8ScX2nqh47o9knUJkk%2FLWNbdAGpR%2Bk8%3D&jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc5MjAwNjg1MCwibmJmIjoxNzkyMDA1MDUwLCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.l2uxOl4nRQFFS7gSpWvSUFiRv0GNjihiLWfj4bjLlxM&response-content-disposition=attachment%3B%20filename%3DTorrServer-linux-amd64&response-content-type=application%2Foctet-stream" \
    -o "${TOOL_DIR}data/ts/TorrServer-linux"

#      "--proxy-server=socks5://host:port",
cat <<EOF | tee "${TOOL_CFG_FILE}"
{
  "listen": {
    "version": true,
    "ip": "0.0.0.0",
    "port": ${TOOL_PORT},
    "scheme": "http",
    "localhost": "127.0.0.1"
  },
  "lowMemoryMode": false,
  "BaseModule": {
    "SkipModules": [
      "Catalog",
      "DLNA",
      "sisi",
      "Sync",
      "TimeCode",
      "Tracks",
      "WebLog"
    ]
  },
  "chromium": {
    "enable": true,
    "executablePath": "/usr/bin/chromium",
    "args": [
      "--disable-blink-features=AutomationControlled",
    ]
  },
    "gst": {
        "aac_bitrate": 192,
        "aac_channels": 2,
        "aac_samplerate": 48000,
        "enable": true,
        "hardwareAcceleration": true,
        "hdr_to_sdr": false,
        "maxTasks": 0,
        "segment_seconds": 9,
        "subtitles": true,
        "transcodeAV1": true,
        "transcodeAVI": false,
        "transcodeH264": true,
        "transcodeH265": true,
        "transcodeVP8": true,
        "transcodeVP9": true,
        "useGpu": false,
        "video_bitrate": 14000,
        "x264Ultrafast": false
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
    "enable": false
  }
}
EOF
# nano "${TOOL_CFG_FILE}"

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

cat <<EOF | tee "${TOOL_DIR}transcoding_manifest.json"
{
  "enable": true,
  "tree": [
    "Services",
    "Controller.cs",
    "ModInit.cs",
    "ModuleConf.cs"
  ]
}
EOF

cat <<EOF | tee "${TOOL_DIR}torrserver_settings.json"
{
  "BitTorr": {
    "CacheSize": 4294967296,
    "ConnectionsLimit": 30,
    "DisableDHT": false,
    "DisablePEX": false,
    "DisableTCP": false,
    "DisableUPNP": false,
    "DisableUTP": false,
    "DisableUpload": false,
    "DownloadRateLimit": 0,
    "EnableDLNA": false,
    "EnableDebug": false,
    "EnableIPv6": true,
    "EnableRutorSearch": true,
    "ForceEncrypt": true,
    "FriendlyName": "",
    "PeersListenPort": 0,
    "PreloadCache": 0,
    "ReaderReadAHead": 95,
    "RemoveCacheOnDrop": false,
    "ResponsiveMode": false,
    "RetrackersMode": 1,
    "SslCert": "",
    "SslKey": "",
    "SslPort": 0,
    "TorrentDisconnectTimeout": 180,
    "TorrentsSavePath": "",
    "UploadRateLimit": 0,
    "UseDisk": false
  }
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
docker container rm -f "\${TOOL_NAME}" 2> /dev/null
docker run \\
    --env "TOOL_CFG_FILE=\${TOOL_CFG_FILE}" \\
    --env "TOOL_DIR=\${TOOL_DIR}" \\
    --env "TOOL_PORT=\${TOOL_PORT}" \\
    --name "\${TOOL_NAME}" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/tcp" \\
    --publish "\${TOOL_PORT}:\${TOOL_PORT}/udp" \\
    --rm \\
    --volume "\${TOOL_CFG_FILE}:/lampac/init.conf" \\
    --volume "\${TOOL_PASSWD_FILE}:/lampac/passwd" \\
    --volume "\${TOOL_DIR}admin_panel_manifest.json:/lampac/module/AdminPanel/manifest.json" \\
    --volume "\${TOOL_DIR}torrserver_settings.json:/lampac/module/TorrServer/settings.json" \\
    --volume "\${TOOL_DIR}transcoding_manifest.json:/lampac/module/Transcoding/manifest.json" \\
    --volume "\${TOOL_DIR}data/ts:/lampac/data/ts" \\
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
