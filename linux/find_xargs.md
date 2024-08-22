# A `find | xargs` multi-threaded pipeline example

```shell script
find "${SRC_DIR}" \
    -name '*.txt' \
    -type f \
    -print0 \
| xargs \
    --max-procs "$(nproc)" \
    --null \
    --replace="{}" \
        bash -c '
            FILE="{}";
            echo "Found text file: ${FILE}";
        '
```
