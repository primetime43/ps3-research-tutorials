# Part 2: Capturing RPCS3 Traffic (Localhost)

## Overview
When using RPCS3, you can redirect game server traffic to localhost (127.0.0.1) for testing custom servers. This captures plaintext traffic which is easier to analyze than encrypted real PS3 traffic.

---

## Step 1: Configure RPCS3 Network Settings

### Global Settings
**In RPCS3:** Configuration → Network tab

### Per-Game Settings (Recommended)
1. Right-click game in RPCS3
2. **Create Custom Configuration**
3. Go to **Network** tab

### Settings Table

| Setting | Value | Description |
|---------|-------|-------------|
| Network Status | Connected | Enable networking |
| PSN Status | RPCN | Required for most online games |
| DNS | 8.8.8.8 | Or your preferred DNS |
| IP/Hosts switches | See below | Domain redirects |
| Bind address | 0.0.0.0 | Listen on all interfaces |
| UPNP | Enabled | For NAT traversal |

---

## Step 2: Configure IP/Hosts Switches

### Format
```
domain1.example.com=<IP>&&domain2.example.com=<IP>
```

### Examples
```
# Single domain redirect to localhost
game.server.com=127.0.0.1

# Multiple domains
auth.server.com=127.0.0.1&&lobby.server.com=127.0.0.1&&stun.server.com=127.0.0.1

# Redirect to custom server IP
game.server.com=192.168.1.100
```

### Finding Domains to Redirect
1. Run game without redirects first
2. Check RPCS3 log for DNS queries (see Part 6)
3. Add discovered domains to IP/Hosts switches

---

## Step 3: Install Loopback Capture Support

### Windows (Npcap)
1. Download Npcap from https://npcap.com
2. During installation, check **"Install Npcap in WinPcap API-compatible Mode"**
3. Check **"Support loopback traffic"**
4. Complete installation
5. Restart Wireshark

### Linux
Loopback capture works by default on the `lo` interface.

### macOS
Loopback capture works by default on the `lo0` interface.

---

## Step 4: Capture with Wireshark

1. Open **Wireshark**
2. Select **Npcap Loopback Adapter** (Windows) or **lo** (Linux/macOS)
3. Click **Start Capture**
4. Launch game in RPCS3 → Go online
5. Perform the actions you want to capture
6. Click **Stop Capture**
7. Save: **File** → **Save As**

---

## Step 5: Filter Localhost Traffic

### Basic Filters
```
# All loopback traffic
ip.addr == 127.0.0.1

# Specific port
tcp.port == 3074 and ip.addr == 127.0.0.1

# Multiple ports
(tcp.port == 3074 or tcp.port == 3075 or tcp.port == 80) and ip.addr == 127.0.0.1

# Exclude specific port (reduce noise)
ip.addr == 127.0.0.1 and not tcp.port == 8080
```

### Direction-Based Filters
```
# Traffic from game to server (requests)
ip.src == 127.0.0.1 and tcp.dstport == 3074

# Traffic from server to game (responses)
ip.dst == 127.0.0.1 and tcp.srcport == 3074
```

---

## Step 6: Understanding RPCS3 Network Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Game     │     │   RPCS3     │     │   Your      │
│  (PS3 Code) │────▶│  DNS Hook   │────▶│   Server    │
│             │     │             │     │ (127.0.0.1) │
└─────────────┘     └─────────────┘     └─────────────┘
      │                    │                    │
      │ 1. DNS Query       │                    │
      │───────────────────▶│                    │
      │                    │ 2. Redirected IP   │
      │◀───────────────────│                    │
      │                    │                    │
      │ 3. Connect to 127.0.0.1:PORT           │
      │────────────────────────────────────────▶│
      │                    │                    │
      │ 4. Send/Receive Data                   │
      │◀───────────────────────────────────────▶│
```

---

## PSN Status Options

| Status | Description | Use Case |
|--------|-------------|----------|
| **RPCN** | Connected to RPCS3's network | Required for most online games |
| **Simulated** | Fake/local PSN | Testing (may crash some games) |
| **Disconnected** | Offline mode | Single-player only |

### Note on PSN Status
Some games **require RPCN** and will crash with "Simulated" mode:
```
RPCN is required to use PSN online features.
```
If you see this error, change PSN Status back to RPCN.

---

## Troubleshooting

### No Loopback Adapter in Wireshark
- Reinstall Npcap with loopback support enabled
- Restart Wireshark after installation
- Run Wireshark as Administrator

### Traffic Not Being Redirected
- Verify IP/Hosts switches format is correct
- Check RPCS3 log for "DnsHook" entries
- Ensure domain spelling matches exactly

### Connection Refused
- Make sure your server is running
- Verify server is listening on correct port
- Check firewall settings

### Game Crashes on "Simulated" PSN
- Change PSN Status to "RPCN"
- Some games require real PSN authentication
