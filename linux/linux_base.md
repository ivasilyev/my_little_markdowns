# Linux basic system scripts

## Update software

```shell script
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get clean; sudo apt-get autoclean; sudo apt-get autoremove -y
```

## Reboot machine

```shell script
sudo shutdown -r now
```

## Get CPU features

### View CPU threads number

```shell script
nproc
grep -c '^processor' "/proc/cpuinfo"
```

## View CPU temperature

```shell script
sudo apt-get install \
    --yes \
    hddtemp \
    lm-sensors

watch sensors
```

## Change host name

```shell script
export HOSTNAME="nanokvm-5"
cat << EOF | tee /etc/fstab
127.0.0.1 localhost
127.0.1.1 ${HOSTNAME}
EOF
cat << EOF | tee 
${HOSTNAME}
EOF

shutdown -r now
```