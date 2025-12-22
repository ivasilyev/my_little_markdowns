# uTorrent & BitTorrent performance setup

1. Open an uTorrent window;
2. Open Options - Preferences;

```yaml
- Connection
  - Listening Port
    - Port used for incoming connections: 50000 - 65000
    # use a direct port forward on your router firewall instead
    - UPnP Port Mapping Check: false
    - Add uTorrent to Windows Firewall exceptions: true
- Bittorrent
  - Basic Bittorrent Features
    - Global maxim number of connections: 500
    - Maximum number of connected peers per torrent: 200
    - Number of upload slots per torrent: 200
    - Enable DHT Network: false
    - Enable DHT for new torrents: false
    - Enable Peer Exchange: false
    - Protocol Encryption
      - Outgoing: Enabled
      - Allow incoming legacy connections: true
- Queueing
  - Queue Settings
    - Maximum number of active torrents: 100
    - Maximum number of active downloads: 100
- Advanced
  - diskio.sparse_files: true
  - net.low_cpu: false
  - net.max_halfopen: 43
  - peer.disconnect_inactive_intervall: 500
  - peer.lazy_bitfield: true
```

# Enable sequential download

1. Open an uTorrent window;
2. Hold down the key combination `Shift` + `F2`, **do not release the keys**;
3. Open Options - Preferences;
 
```yaml
- Advanced
  # sequential download of parts of the files
  # useful to watch movies while they are being downloaded
  - bt.sequential_download: true
  # sequential download of files in the torrent list
  # the file series will be downloaded in order, starting from the first
  - bt.sequential_files: true
```
