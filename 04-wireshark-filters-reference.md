# Part 4: Wireshark Filters Reference

## Overview
A comprehensive reference for Wireshark display filters useful for analyzing game network traffic.

---

## Basic Filters

### By IP Address
```
# Any traffic involving IP
ip.addr == 192.168.1.100

# Source IP only
ip.src == 192.168.1.100

# Destination IP only
ip.dst == 192.168.1.100

# IP range (CIDR notation)
ip.addr == 192.168.1.0/24

# Exclude IP
not ip.addr == 192.168.1.1
!(ip.addr == 192.168.1.1)
```

### By Port
```
# TCP port (source or destination)
tcp.port == 3074

# TCP destination port only
tcp.dstport == 3074

# TCP source port only
tcp.srcport == 3074

# UDP port
udp.port == 3074

# Port range
tcp.port >= 3000 and tcp.port <= 4000

# Multiple ports
tcp.port == 80 or tcp.port == 443 or tcp.port == 3074
tcp.port in {80, 443, 3074}
```

### By Protocol
```
# Common protocols
tcp
udp
dns
http
tls
icmp

# HTTP/2
http2

# Specific application protocols
mysql
ssh
ftp
```

---

## DNS Filters

```
# All DNS traffic
dns

# DNS queries only (not responses)
dns.flags.response == 0

# DNS responses only
dns.flags.response == 1

# Query for specific domain
dns.qry.name == "example.com"

# Query containing text
dns.qry.name contains "game"
dns.qry.name contains ".net"

# DNS response with specific IP (A record)
dns.a == 192.168.1.100

# DNS response with specific IPv6 (AAAA record)
dns.aaaa == ::1

# Specific record types
dns.qry.type == 1      # A record
dns.qry.type == 28     # AAAA record
dns.qry.type == 5      # CNAME
dns.qry.type == 15     # MX

# Failed DNS queries (NXDOMAIN)
dns.flags.rcode == 3
```

---

## TCP Analysis

### Connection State
```
# SYN packets (connection initiation)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# SYN-ACK packets (connection accepted)
tcp.flags.syn == 1 and tcp.flags.ack == 1

# FIN packets (connection closing)
tcp.flags.fin == 1

# RST packets (connection reset)
tcp.flags.reset == 1

# ACK only packets
tcp.flags.ack == 1 and tcp.len == 0

# Packets with data
tcp.len > 0
```

### TCP Problems
```
# Retransmissions
tcp.analysis.retransmission

# Duplicate ACKs
tcp.analysis.duplicate_ack

# Zero window
tcp.analysis.zero_window

# Out of order
tcp.analysis.out_of_order

# Lost segment
tcp.analysis.lost_segment

# Fast retransmission
tcp.analysis.fast_retransmission
```

### TCP Stream
```
# Follow specific TCP stream
tcp.stream eq 0
tcp.stream eq 1

# Multiple streams
tcp.stream eq 0 or tcp.stream eq 1
```

---

## Packet Content Filters

### Contains Bytes (Hex)
```
# TCP payload contains specific bytes
tcp contains 00:12:34:56

# Any frame contains bytes
frame contains 00:12:34:56

# UDP payload contains bytes
udp contains 00:12:34:56
```

### Contains String
```
# Frame contains ASCII string
frame contains "AUTH"
frame contains "password"

# TCP payload contains string
tcp contains "HTTP"
tcp contains "GET"

# Case-insensitive (use matches with regex)
frame matches "(?i)auth"
```

### Payload Length
```
# TCP packets with specific payload size
tcp.len == 303

# TCP packets with payload in range
tcp.len >= 100 and tcp.len <= 500

# UDP packets with specific size
udp.length == 100
```

---

## Combining Filters

### Logical Operators
```
# AND
ip.addr == 192.168.1.100 and tcp.port == 3074

# OR
tcp.port == 80 or tcp.port == 443

# NOT
not dns
!(tcp.port == 22)

# Complex combinations
(ip.addr == 192.168.1.100 or ip.addr == 192.168.1.101) and tcp.port == 3074

# Exclude multiple
not (dns or arp or icmp)
```

### Parentheses for Grouping
```
# Traffic to specific IP on specific ports
ip.dst == 192.168.1.100 and (tcp.port == 80 or tcp.port == 443)

# Either of two complete conditions
(ip.src == 192.168.1.1 and tcp.dstport == 80) or (ip.src == 192.168.1.2 and tcp.dstport == 443)
```

---

## Time-Based Filters

```
# Packets after specific time
frame.time >= "2024-01-15 10:00:00"

# Packets in time range
frame.time >= "2024-01-15 10:00:00" and frame.time <= "2024-01-15 11:00:00"

# Relative time (seconds since capture start)
frame.time_relative >= 10 and frame.time_relative <= 60

# Delta time (time since previous packet)
frame.time_delta > 1
```

---

## TLS/SSL Filters

```
# All TLS traffic
tls

# TLS Client Hello (contains SNI - server name)
tls.handshake.type == 1

# TLS Server Hello
tls.handshake.type == 2

# Extract SNI (Server Name Indication)
tls.handshake.extensions_server_name contains "example"

# Specific TLS version
tls.record.version == 0x0303  # TLS 1.2

# TLS alerts
tls.alert_message
```

---

## HTTP Filters

```
# All HTTP traffic
http

# HTTP requests
http.request

# HTTP responses
http.response

# Specific method
http.request.method == "GET"
http.request.method == "POST"

# Specific URI
http.request.uri contains "/api"

# Specific host
http.host contains "example.com"

# Response codes
http.response.code == 200
http.response.code >= 400  # Errors

# Content type
http.content_type contains "json"
```

---

## Useful Filter Combinations

### Game Traffic Analysis
```
# All traffic to/from game server
ip.addr == <SERVER_IP> and (tcp.port == 3074 or udp.port == 3074)

# Connection attempts to server
ip.dst == <SERVER_IP> and tcp.flags.syn == 1 and tcp.flags.ack == 0

# Data packets only (no handshake/ack)
ip.addr == <SERVER_IP> and tcp.len > 0

# First packet of each connection
tcp.flags.syn == 1 and tcp.flags.ack == 0
```

### Localhost Testing
```
# All loopback traffic on specific port
ip.addr == 127.0.0.1 and tcp.port == 3074

# Requests to local server
ip.dst == 127.0.0.1 and tcp.dstport == 3074 and tcp.len > 0

# Responses from local server
ip.src == 127.0.0.1 and tcp.srcport == 3074 and tcp.len > 0
```

### Troubleshooting
```
# All errors and resets
tcp.flags.reset == 1 or tcp.analysis.retransmission

# Failed connections
tcp.flags.reset == 1 and tcp.flags.ack == 1

# DNS failures
dns.flags.rcode != 0
```

---

## Quick Reference Card

| Purpose | Filter |
|---------|--------|
| All DNS | `dns` |
| Specific port | `tcp.port == 3074` |
| Specific IP | `ip.addr == 192.168.1.100` |
| Connection attempts | `tcp.flags.syn == 1 and tcp.flags.ack == 0` |
| Data packets | `tcp.len > 0` |
| Contains string | `frame contains "AUTH"` |
| Contains hex | `tcp contains 00:12:34` |
| Localhost | `ip.addr == 127.0.0.1` |
| Exclude DNS | `not dns` |
| Follow stream | `tcp.stream eq 0` |
