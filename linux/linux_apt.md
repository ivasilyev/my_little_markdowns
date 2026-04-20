# Linux AutoPackageTool commands

## Update software

```shell script
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get autoremove -y
```

## Update software via a SOCKS5 proxy

```shell script
export PROXY_URL_STRING="127.0.0.1:1080"
sudo apt-get \
    -o Acquire::http::Proxy="socks5h://${PROXY_URL_STRING}" \
    -o Acquire::https::Proxy="socks5h://${PROXY_URL_STRING}" \
    update
```

## Remove package

```shell script
sudo apt remove \
    --purge \
    --yes package
```
