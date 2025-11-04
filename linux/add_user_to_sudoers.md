# Add Linux user to sudoers group

Run by regular user:

```shell script
sudo adduser "$(whoami)" sudo
sudo nano /etc/sudoers
```
```shell script
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL) NOPASSWD: ALL
```
```shell script
# The changes will take effect at the next login
logout
```
