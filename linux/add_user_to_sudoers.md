# Add Linux user to sudoers group

Run by regular user:

```
sudo adduser "$(whoami)" sudo
sudo nano /etc/sudoers
```
```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL) NOPASSWD: ALL
```
```
# The changes will take effect at the next login
logout
```
