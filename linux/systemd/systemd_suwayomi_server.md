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

## Create auto update script

Usually a batch update fails, so the iterative approach seems to be required. 

```shell script
export UPDATE_SCRIPT="${TOOL_DIR}auto_update.sh"
export SERVER_URL="http://127.0.0.1:4567"
export AUTH_FLAGS=""

touch "${UPDATE_SCRIPT}" && chmod a+x "${UPDATE_SCRIPT}"

echo "0 5 * * * \"${UPDATE_SCRIPT}\" \"${SERVER_URL}\" 2>&1 &"
# crontab -e

nano "${UPDATE_SCRIPT}"
```

```shell script
#!/usr/bin/env bash

export SERVER_URL="${1}"
export AUTH_FLAGS="${2}"
export GRAPHQL_URL="${SERVER_URL}/api/graphql"

# echo "Cancel stalled downloads"
# curl -s -X POST \
#     -H "Content-Type: application/json" \
#     -d '{"operationName":"CLEAR_DOWNLOADER","variables":{"input":{}},"query":"fragment DOWNLOAD_TYPE_FIELDS on DownloadType { chapter { id name sourceOrder isDownloaded __typename } manga { id title downloadCount __typename } progress state tries __typename } fragment DOWNLOAD_STATUS_FIELDS on DownloadStatus { state queue { ...DOWNLOAD_TYPE_FIELDS __typename } __typename } mutation CLEAR_DOWNLOADER($input: ClearDownloaderInput = {}) { clearDownloader(input: $input) { downloadStatus { ...DOWNLOAD_STATUS_FIELDS __typename } __typename } }"}' \
#     "${GRAPHQL_URL}" \
#     > /dev/null 2>&1

echo "Get total entries count"
TOTAL_MANGA_ENTRIES_COUNT="$(
    curl -X POST \
        -H "Content-Type: application/json" \
        -d '{"operationName": "GET_LIBRARY_MANGA_COUNT", "variables": {}, "query": "query GET_LIBRARY_MANGA_COUNT {mangas(condition: {inLibrary: true}) {totalCount __typename}}"}' \
        "${GRAPHQL_URL}" \
    | jq '.data.mangas.totalCount'
)"

echo "Update ${TOTAL_MANGA_ENTRIES_COUNT} entries"
for ENTRY_ID in $(seq ${TOTAL_MANGA_ENTRIES_COUNT})
    do
    echo "Update entry ${ENTRY_ID}"
    ENTRY_JSON="$(
        curl -s -X POST \
            -H "Content-Type: application/json" \
            -d '{"operationName":"REFRESH_MANGA","variables":{"id":'${ENTRY_ID}'},"query":"fragment MANGA_BASE_FIELDS on MangaType { id title thumbnailUrl thumbnailUrlLastFetched inLibrary initialized sourceId __typename } fragment MANGA_CHAPTER_STAT_FIELDS on MangaType { id unreadCount downloadCount bookmarkCount hasDuplicateChapters chapters { totalCount __typename } __typename } fragment MANGA_CHAPTER_NODE_FIELDS on MangaType { firstUnreadChapter { id sourceOrder isRead mangaId chapterNumber name scanlator __typename } lastReadChapter { id sourceOrder lastReadAt __typename } latestReadChapter { id sourceOrder lastReadAt __typename } latestFetchedChapter { id fetchedAt __typename } latestUploadedChapter { id uploadDate __typename } highestNumberedChapter { id chapterNumber __typename } __typename } fragment MANGA_META_FIELDS on MangaMetaType { mangaId key value __typename } fragment SOURCE_BASE_FIELDS on SourceType { id name displayName lang iconUrl __typename } fragment MANGA_LIBRARY_FIELDS on MangaType { ...MANGA_BASE_FIELDS ...MANGA_CHAPTER_STAT_FIELDS ...MANGA_CHAPTER_NODE_FIELDS genre lastFetchedAt inLibraryAt status artist author description meta { ...MANGA_META_FIELDS __typename } source { ...SOURCE_BASE_FIELDS __typename } trackRecords { totalCount nodes { id trackerId __typename } __typename } __typename } fragment MANGA_MIGRATION_FIELDS on MangaType { ...MANGA_BASE_FIELDS ...MANGA_CHAPTER_NODE_FIELDS artist author source { id name displayName __typename } __typename } fragment MANGA_SCREEN_FIELDS on MangaType { ...MANGA_LIBRARY_FIELDS ...MANGA_CHAPTER_NODE_FIELDS ...MANGA_MIGRATION_FIELDS artist author description status realUrl meta { ...MANGA_META_FIELDS __typename } sourceId source { id name displayName __typename } trackRecords { totalCount nodes { id trackerId __typename } __typename } __typename } fragment CHAPTER_BASE_FIELDS on ChapterType { id name mangaId scanlator realUrl sourceOrder chapterNumber __typename } fragment CHAPTER_STATE_FIELDS on ChapterType { id isRead isDownloaded isBookmarked __typename } fragment CHAPTER_LIST_FIELDS on ChapterType { ...CHAPTER_BASE_FIELDS ...CHAPTER_STATE_FIELDS fetchedAt uploadDate lastReadAt __typename } mutation REFRESH_MANGA($id: Int!) { fetchChapters(input: {mangaId: $id}) { chapters { ...CHAPTER_LIST_FIELDS __typename } __typename } fetchManga(input: {id: $id}) { manga { ...MANGA_SCREEN_FIELDS __typename } __typename } }"}' \
            "${GRAPHQL_URL}" \
        2> /dev/null
    )"
    ENTRY_TITLE="$(
        echo "${ENTRY_JSON}" |  jq -r '.data.fetchManga.manga.title'
    )"
    CHAPTER_IDS="$(
        echo "${ENTRY_JSON}" | jq -r '[.data.fetchChapters.chapters.[].id | tostring] | join(",")' \
        2> /dev/null
    )"
    if [[ -n "${CHAPTER_IDS}" ]]
        then
        echo "Download chapters for '${ENTRY_TITLE}': ${CHAPTER_IDS}"
        curl -s -X POST \
            -H "Content-Type: application/json" \
            -d '{"operationName":"ENQUEUE_CHAPTER_DOWNLOADS","variables":{"input":{"ids":['"${CHAPTER_IDS}"']}},"query":"fragment DOWNLOAD_TYPE_FIELDS on DownloadType { chapter { id name sourceOrder isDownloaded __typename } manga { id title downloadCount __typename } progress state tries __typename } fragment DOWNLOAD_STATUS_FIELDS on DownloadStatus { state queue { ...DOWNLOAD_TYPE_FIELDS __typename } __typename } mutation ENQUEUE_CHAPTER_DOWNLOADS($input: EnqueueChapterDownloadsInput!) { enqueueChapterDownloads(input: $input) { downloadStatus { ...DOWNLOAD_STATUS_FIELDS __typename } __typename } }"}' \
            "${GRAPHQL_URL}" > /dev/null 2>&1
        fi
    sleep 1
    done

echo "Done"
```
