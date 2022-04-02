# Torrent Analyze

(There is also a cooler way of doing this which is to reconstruct the torrents)

Looking at the capture/logs, we can see that there are several BitTorrent handshakes.

```
No.     Time           Source                Destination           Protocol Length Info
     79 4.864344025    5.189.157.90          192.168.73.132        BT-DHT   139    BitTorrent DHT Protocol

Frame 79: 139 bytes on wire (1112 bits), 139 bytes captured (1112 bits) on interface eth0, id 0
Ethernet II, Src: VMware_f5:e4:05 (00:50:56:f5:e4:05), Dst: VMware_2d:4b:5e (00:0c:29:2d:4b:5e)
Internet Protocol Version 4, Src: 5.189.157.90, Dst: 192.168.73.132
User Datagram Protocol, Src Port: 12023, Dst Port: 51413
BitTorrent DHT Protocol
    Request arguments: Dictionary...
        Key: a
        Value: Dictionary...
            id: 99a047b561fc2b3e22dedcc23a7a237fcbd6c47b
                Key: id
                Value: 99a047b561fc2b3e22dedcc23a7a237fcbd6c47b
            info_hash: 17d62de1495d4404f6fb385bdfd7ead5c897ea22
                Key: info_hash
                Value: 17d62de1495d4404f6fb385bdfd7ead5c897ea22
            Terminator: e
    Request type: get_peers
        Key: q
        Value: get_peers
    Transaction ID: ce83d90f
        Key: t
        Value: ce83d90f
    Message type: Request
        Key: y
        Value: q
    Terminator: e
```

Turn these info hashes into magnet links (or google them):

```
magnet:?xt=urn:btih:17d62de1495d4404f6fb385bdfd7ead5c897ea22    
magnet:?xt=urn:btih:17c1e42e811a83f12c697c21bed9c72b5cb3000d    
magnet:?xt=urn:btih:d59b1ce3bf41f1d282c1923544629062948afadd    
magnet:?xt=urn:btih:078e18df4efe53eb39d3425e91d1e9f4777d85ac    
magnet:?xt=urn:btih:7af6be54c2ed4dcb8d17bf599516b97bb66c0bfd    
magnet:?xt=urn:btih:17c0c2c3b7825ba4fbe2f8c8055e000421def12c    
magnet:?xt=urn:btih:17c02f9957ea8604bc5a04ad3b56766a092b5556    
magnet:?xt=urn:btih:e2467cbf021192c241367b892230dc1e05c0580e
```

Downloading from these magnet links will show the filename of the file being downloaded,
which is `ubuntu-19.10-desktop-amd64.iso`, so our flag is `picoCTF{ubuntu-19.10-desktop-amd64.iso}`.
