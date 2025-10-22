
# Install MikroTik WinBox GUI on Linux

Run by regular user:

```
sudo apt-get update -y
sudo apt-get install \
    --yes \
    libxcb-icccm4 \
    libxcb-image0 \
    libxcb-keysyms1 \
    libxcb-render-util0 \

export TOOL_NAME="winbox"
export TOOL_DIRECTORY="/opt/${TOOL_NAME}/"
export TOOL_BIN="${TOOL_DIRECTORY}${TOOL_NAME}"
export TMP_FILE="/tmp/WinBox_Linux.zip"
curl \
    -fsSL "https://download.mikrotik.com/routeros/winbox/4.0beta35/WinBox_Linux.zip" \
    -o "${TMP_FILE}"
unzip \
    -o \
    -d "/tmp/WinBox_Linux" \
    "${TMP_FILE}"
    
sudo rm -rfv "${TOOL_DIRECTORY}"
sudo mkdir -p "${TOOL_DIRECTORY}"
sudo mv -v \
    "/tmp/WinBox_Linux/WinBox" \
    "${TOOL_BIN}"
sudo chmod -Rv a+x "${TOOL_DIRECTORY}"
ln -s \
    "${TOOL_BIN}" \
    "${HOME}/Desktop/${TOOL_NAME}"
```
