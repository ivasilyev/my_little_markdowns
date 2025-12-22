# Set up OpenWRT ACL

## Using Linux ACL

```shell script
echo Export variables
export UN=""
export UPW=""

echo Update software
opkg update
opkg install \
    shadow-groupadd \
    shadow-chpasswd \
    shadow-useradd \
    shadow-userdel \
    sudo

echo Create a new user by running useradd
userdel "${UN}"
rm -rf "/home/${UN}"
mkdir -p /home
useradd -m -b /home -g 100 -s /bin/ash "${UN}"

echo Set the password
echo "${UN}:${UPW}" | sudo chpasswd

echo Create the sudo group
groupadd sudo

echo Add user to sudo group
usermod -a -G sudo "${UN}"

echo Edit the sudoers file to allow user of group sudo to become root
vi /etc/sudoers

# remove the # from the line: # %sudo ALL=(ALL) ALL
# close and save the file

# Try to login with the new user to confirm its working
# Ensure that ROOT can not be used as login with SSH in the future:
vi /etc/config/sshd

# add a # in front of the line:
# option PermitRootLogin yes

# close and save the file

# Restart the sshd
/etc/init.d/sshd restart

# Done - now only the user can login
# To run a command with root rights use
# sudo <command>
```

## Using `luci-app-multi-user` package (recommended)

```shell script
opkg update
opkg install \
    luci-app-acl \
    luci-app-multi-user
```

