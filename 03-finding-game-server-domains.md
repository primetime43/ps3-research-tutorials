# Part 3: Finding Game Server Domains

## Overview
Before you can redirect game traffic, you need to identify what domains/servers the game connects to. This guide covers multiple methods to discover these domains.

---

## Method 1: DNS Query Analysis (Wireshark)

### Step-by-Step
1. Set up traffic capture (see Part 1 or 2)
2. Start Wireshark capture
3. Launch game and attempt to go online
4. Stop capture

### Filter for DNS
```
# All DNS queries
dns

# Only DNS queries (not responses)
dns.flags.response == 0

# DNS queries containing specific text
dns.qry.name contains "game"
dns.qry.name contains "server"
dns.qry.name contains "auth"
dns.qry.name contains "lobby"
```

### Identify Domains
1. Look at the **Info** column for query names
2. Note down all unique domains
3. Check DNS responses for resolved IPs:
```
# Find what IP a domain resolves to
dns.qry.name == "example.server.com"
```

### Common Domain Patterns
```
auth.*.com          # Authentication servers
lobby.*.com         # Matchmaking/lobby
stun.*.com          # NAT traversal
master.*.com        # Master server lists
stats.*.com         # Statistics/leaderboards
storage.*.com       # Cloud saves/data
cdn.*.com           # Content delivery
```

---

## Method 2: RPCS3 Log Analysis

### Log Location
```
Windows: C:\Users\<username>\Documents\Emulators\PS3\RPCS3\log\RPCS3.log
Linux:   ~/.config/rpcs3/log/RPCS3.log
macOS:   ~/Library/Application Support/rpcs3/log/RPCS3.log
```

### Search Commands

**Windows (PowerShell):**
```powershell
# Find DNS queries
Select-String -Path "RPCS3.log" -Pattern "DnsHook"

# Find connection attempts
Select-String -Path "RPCS3.log" -Pattern "Attempting to connect"

# Find all network activity
Select-String -Path "RPCS3.log" -Pattern "sys_net"
```

**Linux/macOS:**
```bash
# Find DNS queries
grep -i "DnsHook" RPCS3.log

# Find connection attempts
grep -i "Attempting to connect" RPCS3.log

# Find all network activity
grep -i "sys_net" RPCS3.log

# Find domain-like strings
grep -oE "[a-zA-Z0-9.-]+\.(com|net|org)" RPCS3.log | sort -u
```

### Example Log Entries
```
DnsHook: DNS query for game.server.example.com
DnsHook: Solving game.server.example.com to 127.0.0.1
sys_net: [Native] Attempting to connect on 192.168.1.100:3074
```

---

## Method 3: Static Analysis (Game Executable)

### Extract Strings from EBOOT.BIN

**Location:**
```
<GAME_FOLDER>/PS3_GAME/USRDIR/EBOOT.BIN
```

**Using `strings` command:**
```bash
# Find all potential domains
strings EBOOT.BIN | grep -iE "\.(com|net|org|io)"

# Find URLs
strings EBOOT.BIN | grep -iE "https?://"

# Find specific patterns
strings EBOOT.BIN | grep -i "server"
strings EBOOT.BIN | grep -i "auth"
strings EBOOT.BIN | grep -i "lobby"
```

**Using Ghidra:**
1. Open EBOOT.BIN in Ghidra
2. Window → Defined Strings
3. Filter for `.com`, `.net`, etc.

---

## Method 4: Network Monitor Tools

### Windows Resource Monitor
1. Open **Resource Monitor** (resmon.exe)
2. Go to **Network** tab
3. Run game in RPCS3
4. Watch **Network Activity** section for connections

### TCPView (Windows)
1. Download TCPView from Microsoft Sysinternals
2. Run it alongside RPCS3
3. Filter by rpcs3.exe process
4. Watch for new connections

### netstat
```bash
# Windows - watch for new connections
netstat -an | findstr ESTABLISHED

# Linux - watch connections in real-time
watch -n 1 'netstat -an | grep ESTABLISHED'
```

---

## Method 5: Packet Inspection

### Look for Domain Strings in Packets
```
# Wireshark filter for packets containing domain text
frame contains "server"
frame contains ".com"
frame contains ".net"
```

### Check TLS/SSL Handshakes
Even encrypted traffic reveals domains in SNI (Server Name Indication):
```
# Filter for TLS Client Hello (contains SNI)
tls.handshake.type == 1
```

---

## Documenting Discovered Domains

### Create a Domain List
```markdown
| Domain | IP | Port | Purpose |
|--------|-----|------|---------|
| auth.game.com | 192.168.1.100 | 3074 | Authentication |
| lobby.game.com | 192.168.1.100 | 3074 | Matchmaking |
| stun.game.com | 192.168.1.101 | 3478 | NAT Traversal |
```

### Build IP/Hosts Switch String
```
auth.game.com=127.0.0.1&&lobby.game.com=127.0.0.1&&stun.game.com=127.0.0.1
```

---

## Common Server Types

| Type | Common Names | Typical Ports |
|------|--------------|---------------|
| Authentication | auth, login, account | 80, 443, 3074 |
| Lobby/Matchmaking | lobby, matchmaking, mm | 3074, 3478 |
| Game Server | game, play, dedicated | 27015, 28960 |
| Master Server | master, list, browser | 27010, 28910 |
| STUN/NAT | stun, nat, turn | 3478, 3479 |
| Stats/Leaderboards | stats, rankings | 80, 443 |
| Storage/Saves | storage, cloud, save | 80, 443 |
| Content Delivery | cdn, content, patch | 80, 443 |

---

## Tips

1. **Run game multiple times** - Different servers may be contacted at different stages
2. **Try different game modes** - Campaign vs Multiplayer may use different servers
3. **Check for regional servers** - us., eu., asia. prefixes
4. **Look for backup/fallback servers** - Games often have multiple server addresses
5. **Document the order** - Note which servers are contacted first (usually auth)
