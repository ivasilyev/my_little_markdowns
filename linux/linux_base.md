# Linux basic system scripts

## Update software

```shell script
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt-get autoremove -y
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
