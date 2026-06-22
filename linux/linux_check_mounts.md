# Linux critical mount check

```shell script
# Create control files first (assuming that /data and was already mounted)
TARGET_FILES=(
    "/data/.mount_ready"
)
for FILE in "${TARGET_FILES[@]}"
    do
    echo "Create ${FILE}"
    touch "${FILE}"
    done


export TOOL_SCRIPT="/usr/local/bin/check_all_mounts.sh"

cat <<EOF | sudo tee "${TOOL_SCRIPT}"
#!/bin/bash

# Ensure the script runs with root privileges
if [ "\$EUID" -ne 0 ]; then
    echo "Error: This script must be run as root (sudo)." >&2
    exit 1
fi

# Array of all the target files
TARGET_FILES=(
    "/data/MOUNT_READY"
)


# Loop through and check each file
for FILE in "\${TARGET_FILES[@]}"
    do
    if [ ! -f "\${FILE}" ]; then
        echo "Critical file missing: \${FILE}. Rebooting in 30 seconds..."
        /sbin/shutdown -r +1 "Missing critical mount point verified by \${FILE}. Rebooting in 30 seconds." &
        sleep 30
        exit 0
    fi
    done

echo "All mount points are healthy."
EOF

sudo chmod +x "${TOOL_SCRIPT}"

echo "*/5 * * * * \"${TOOL_SCRIPT}\" > /dev/null 2>&1"

# sudo crontab -e
# sudo crontab -l
```
