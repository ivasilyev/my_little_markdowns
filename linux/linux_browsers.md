# Linux browsers setup

## Install Firefoxes

```shell script
sudo install -d -m 0755 /etc/apt/keyrings

wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null

gpg -n -q --import --import-options import-show /etc/apt/keyrings/packages.mozilla.org.asc | awk '/pub/{getline; gsub(/^ +| +$/,""); if($0 == "35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3") print "\nThe key fingerprint matches ("$0").\n"; else print "\nVerification failed: the fingerprint ("$0") does not match the expected one.\n"}'

cat <<EOF | sudo tee "/etc/apt/sources.list.d/mozilla.sources"
Types: deb
URIs: https://packages.mozilla.org/apt
Suites: mozilla
Components: main
Signed-By: /etc/apt/keyrings/packages.mozilla.org.asc
EOF

echo '
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000
' | sudo tee /etc/apt/preferences.d/mozilla 

sudo apt-get update -y
sudo apt-get install -y \
    firefox \
    firefox-beta \
    firefox-devedition \
    firefox-esr \
    firefox-nightly
```

## Install Chromes (deprecated)

```shell script
# https://www.google.com/linuxrepositories/

wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | sudo tee /etc/apt/trusted.gpg.d/google.asc >/dev/null

wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/google.gpg >/dev/null
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/google-chrome.gpg] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list

sudo apt-get update -y

sudo apt-get install -y \
    google-chrome-stable \
    google-chrome-beta \
    google-chrome-unstable 
```

## Install Chromium instead

```shell script
sudo apt-get install -y chromium-browser
```

## Install Mozilla Thunderbird

```shell script
sudo apt-get install -y thunderbird
```

# Other

## Disable Firefox session restore

- Check Startup Settings: 
  - Go to Firefox Settings (three-bar menu)
  - General
  - Startup
  - `Restore previous session` is unchecked
  - `When Firefox starts` is set to "Show your home page" or "Show a blank page"
- Check `about:config`:
  - Type `about:config` in the address bar
  - Search for `browser.sessionstore.enabled`
  - Set it to `false` to disable session restore entirely
- Check `about:addons` for any tab management extensions that might be overriding settings. 
