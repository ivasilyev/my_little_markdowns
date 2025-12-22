# uTorrent & BitTorrent performance setup

```yaml
- Options 
- Preferences
- Connection
    - Port used for incoming connections: 50000 - 65000
    - Uncheck UPnP Port Mapping Check
    - Add uTorrent to Windows Firewall exceptions
- Bittorrent
    - Global maxim number of connections: 500
    - Maximum number of connected peers per torrent: 200
    - Number of upload slots per torrent: 200
    - Enable DHT Network: false
    - Enable DHT for new torrents: false
    - Enable Peer Exchange: false
    - Outgoing: Enabled
    - Check Allow incoming legacy connections
- Queueing
    - Maximum number of active torrents: 100
    - Maximum number of active downloads: 100
- Advanced
    - diskio.sparse_files: true
    - net.low_cpu: false
    - net.max_halfopen: 43
    - peer.disconnect_inactive_intervall: 500
    - peer.lazy_bitfield: true
```
