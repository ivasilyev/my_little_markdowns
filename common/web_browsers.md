# Tweaks for web browsers

## Firefox

### Disable session restore

- Check Startup Settings: 
  - Go to Firefox Settings (three-bar menu) 
  - General 
  - Startup
  - Uncheck "Restore previous session" is unchecked, 
  - "When Firefox starts" is set to "Show your home page" or "Show a blank page".
- Check about:config:
  - Type `about:config` in the address bar, accept the risk
  - Set `browser.sessionstore.enabled` to `false` 
- Disable Extensions: 
  - Check `about:addons` for any tab management extensions that might be overriding settings. 
