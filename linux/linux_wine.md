# Linux Wine setup

```
echo Install Wine
sudo dpkg --add-architecture i386 && \
sudo apt-get update -y && \
sudo apt-get install \
    --install-recommends \
    --yes \
    wine \
    wine-stable \
    wine32:i386
    # winehq-stable

echo Clear Wine environment
rm -rf ~/.wine

echo Initialize Wine environment
wine notepad

echo Browse Wine C:/ folder
ls ~/.wine/drive_c/
```
