
```shell script
# Start server

export TOOL="iperf"
export URL=""

"${TOOL}" --server --port 5005

# Start client

"${TOOL}" \
    --client "${URL}" \
    --interval 10s \
    --port 5005 \
    --time 60s
```
