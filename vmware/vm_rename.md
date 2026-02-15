# Rename virtual machine in-place

```shell script
cd "/tmp/vm_directory"

export OLD_NAME=""
export NEW_NAME=""

echo "Clean cache"
find \
    "$(pwd)" \
    -name "*.lck" \
    -type d \
    -print0 \
| xargs --null -I {} bash -c '
    rm -rfv "${}"
'

echo "Rename files"
find \
    . \
    -print0 \
    -type f \
| xargs \
    --null \
    --replace={} \
    bash -c '
        OLD_FILE="{}";
        NEW_FILE="$(
            echo "${OLD_FILE}" \
            | sed "s/${OLD_NAME}/${NEW_NAME}/g"
        )";
        mv -v "${OLD_FILE}" "${NEW_FILE}";
    '

echo "Rename content"
for EXT in "vmdk" "vmx" "vmxf"
    do
    FILE="${NEW_NAME}.${EXT}"
    echo "Rename content for '${FILE}'"
    sed --in-place "s/${OLD_NAME}/${NEW_NAME}/g" "${FILE}"
    done
```
