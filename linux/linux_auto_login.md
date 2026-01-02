# Linux user auto login

## `xfce`

```shell script
sudo nano /etc/lightdm/lightdm.conf
```
```text
[Seat:*]
autologin-session=xubuntu
autologin-user=<username>
autologin-user-timeout=0
```
