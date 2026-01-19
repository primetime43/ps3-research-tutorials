# Part 1: Capturing Real PS3 Traffic (Via PC Gateway)

## Prerequisites
- Physical PS3 console
- PC with two network interfaces (WiFi + Ethernet, or two Ethernet ports)
- Wireshark installed on PC
- Ethernet cable to connect PS3 to PC

---

## Step 1: Set Up PC as Gateway (Internet Connection Sharing)

### Windows
1. Open **Network Connections** (`ncpa.cpl`)
2. Right-click your **internet-connected adapter** (e.g., WiFi) → Properties
3. Go to **Sharing** tab
4. Check "Allow other network users to connect through this computer's Internet connection"
5. Select the adapter connected to PS3 (e.g., Ethernet)
6. Click OK

**Result:** Your PC now acts as a router for the PS3

### Linux (Alternative)
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Set up NAT (replace eth0 with internet interface, eth1 with PS3 interface)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

---

## Step 2: Configure PS3 Network

1. On PS3: **Settings** → **Network Settings** → **Internet Connection Settings**
2. Choose **Custom** setup
3. Select **Wired Connection**
4. Set **IP Address** to **Manual**:

| Setting | Value |
|---------|-------|
| IP Address | `192.168.137.2` (or similar in the ICS subnet) |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `192.168.137.1` (your PC's ICS IP) |
| Primary DNS | `192.168.137.1` or `8.8.8.8` |
| Secondary DNS | `8.8.4.4` (optional) |

5. MTU: **Automatic**
6. Proxy Server: **Do Not Use**
7. UPnP: **Enable**
8. Save and test connection

---

## Step 3: Capture Traffic with Wireshark

1. Open **Wireshark** on PC
2. Select the **adapter connected to PS3** (e.g., Ethernet)
3. Click **Start Capture** (blue shark fin icon)
4. On PS3: Launch game → Connect online
5. Wait for connection attempt to complete (or fail)
6. Click **Stop Capture** (red square icon)
7. Save capture: **File** → **Save As** → `capture.pcapng`

---

## Step 4: Filter & Analyze

### Basic Filters
```
# Filter by PS3 IP address
ip.addr == 192.168.137.2

# Filter traffic between PS3 and specific server
ip.addr == 192.168.137.2 and ip.addr == <SERVER_IP>

# Show only outgoing traffic from PS3
ip.src == 192.168.137.2

# Show only incoming traffic to PS3
ip.dst == 192.168.137.2
```

### DNS Analysis
```
# All DNS traffic
dns

# DNS queries from PS3
dns and ip.src == 192.168.137.2

# Find server IP from DNS response
dns.a == <SERVER_IP>

# DNS queries for specific domain
dns.qry.name contains "<PARTIAL_DOMAIN>"
```

### Port-Based Filtering
```
# Filter by specific port
tcp.port == <PORT>
udp.port == <PORT>

# Multiple ports
tcp.port == 80 or tcp.port == 443 or tcp.port == 3074
```

---

## Step 5: Export Captured Data

### Export TCP/UDP Stream
1. Find a packet in the conversation
2. Right-click → **Follow** → **TCP Stream** (or UDP Stream)
3. View the full conversation
4. **Save as** raw data or text

### Export Packet Data
1. **File** → **Export Packet Dissections** → **As Plain Text**
2. Or: **File** → **Export Packet Bytes** (for single packet)

### Export Specific Packets
1. Apply filter to show only relevant packets
2. **File** → **Export Specified Packets**
3. Choose format (pcapng recommended)

---

## Troubleshooting

### PS3 Can't Connect to Internet
- Verify ICS is enabled on correct adapter
- Check PS3 IP settings match ICS subnet (192.168.137.x)
- Ensure gateway IP is correct (192.168.137.1)
- Try disabling Windows Firewall temporarily

### No Traffic Appearing in Wireshark
- Make sure you selected the correct adapter
- Verify PS3 is actually sending traffic (check PS3 connection test)
- Check Wireshark has permission to capture (run as admin)

### Can't See Packet Contents
- Traffic may be encrypted (TLS/SSL)
- Some protocols use custom encryption
- Look for plaintext portions or metadata
