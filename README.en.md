# Netkit CLI Guide

[中文](./README.md) | English

> A binary-distributed network toolbox for day-to-day troubleshooting with a consistent, readable CLI

**Command:** `nk` (short for Netkit)

## Overview

The following are the common command-line workflows for `nk`.

## Quick Start

```bash
# 1) Can the target be reached?
nk ping example.com
nk check tcp example.com 22,80,443

# UDP checks (payload / preset supported)
nk check example.com 53 -P udp --preset dns

# 2) Is DNS correct?
nk dns example.com --type A,AAAA,MX

# 3) Where does the path break?
nk mtr example.com
nk trace example.com

# 4) Local network and interfaces
nk ipinfo              # local network overview
nk iface               # network interface information

# 5) Path MTU discovery
nk mtu example.com     # discover the MTU to the host

# 6) Local discovery
nk mdns --resolve      # discover and resolve mDNS services
```

## Output Format

Netkit uses the global `--output` flag to switch between two output formats:

- **Text mode (default):** color, tables, and visual output
- **JSON mode:** structured JSON for scripts and automation

```bash
# Text output (default)
nk dns example.com

# JSON output
nk --output json dns example.com | jq .

# JSON is supported by all commands
nk --output json iface
nk --output json route
nk --output json neigh
```

## Command Summary

### Core

#### Connectivity
- **`nk ping`** - enhanced ICMP ping with visual output
- **`nk check`** - port reachability checks, similar to `telnet` / `nc`, with `--wait`

#### Name Services
- **`nk dns`** - DNS queries, similar to `dig`

#### Path
- **`nk mtr`** - network path quality monitoring, combining `ping` + `traceroute`
- **`nk trace`** - route tracing with `--enrich`

#### Local Overview
- **`nk ipinfo`** - local network overview, local-only by default
- **`nk iface`** - interface information
- **`nk route`** - routing table
- **`nk neigh`** - ARP / ND neighbor table

### Extended

#### Discovery
- **`nk mdns`** - mDNS / Bonjour service discovery
- **`nk scan`** - port scanning for authorized targets only

#### Security
- **`nk tls`** - TLS / certificate checks
- **`nk whois`** - WHOIS lookup

#### Observability
- **`nk ports`** - local port and process mapping
- **`nk firewall`** - read-only firewall status and rule overview
- **`nk top`** - per-process traffic statistics
- **`nk watch`** - packet capture statistics / live view

#### Devices / Wireless
- **`nk wifi`** - WiFi scan and inspection
- **`nk bt`** - Bluetooth device scan and inspection

#### Tools
- **`nk http`** - HTTP client
- **`nk mtu`** - path MTU discovery
- **`nk diag`** - one-shot diagnostics
- **`nk speedtest`** - speed test
- **`nk iperf`** - iperf3-based bandwidth test

## Command Reference

### HTTP Client

```bash
# GET with custom headers
nk http https://api.example.com/data -H "Authorization: Bearer token" -H "Accept: application/json"

# JSON POST
nk http https://api.example.com/upload -X POST -d '{"name":"test"}' -H "Content-Type: application/json"

# Download with progress
nk http https://example.com/file.zip -o file.zip --progress

# View timing breakdown
nk http https://example.com --timing
```

### DNS Queries

```bash
# Basic query
nk dns example.com

# Query multiple record types
nk dns example.com --type A,AAAA,MX,TXT

# Specify a DNS server
nk dns example.com --server 8.8.8.8

# JSON output
nk --output json dns example.com
```

### Port Checks

```bash
# Single or multiple ports
nk check example.com 80
nk check example.com 22,80,443,8080

# Wait for a service to come up (Docker / CI)
nk check localhost 3000 --wait --wait-timeout 30

# UDP checks
nk check example.com 53 -P udp --preset dns
nk check example.com 123 -P udp --preset ntp

# Custom UDP payload
nk check example.com 9999 -P udp --payload "hello" --expect any
nk check example.com 9999 -P udp --payload-hex deadbeef --timeout 2
```

### Port Scanning

```bash
# Default scan of the first 100 ports
nk scan example.com

# Full port range
nk scan example.com --all
```

### TLS / Certificate Checks

```bash
# Inspect TLS handshake and certificate chain
nk tls example.com:443

# Override SNI
nk tls 1.1.1.1:443 --sni one.one.one.one

# Provide ALPN
nk tls example.com:443 --alpn h2,http/1.1

# Skip verification but still show details
nk tls example.com:443 --insecure

# Hide the certificate chain
nk tls example.com:443 --no-chain
```

### Local Port and Process Mapping

```bash
# Show local ports, PID, and process names
nk ports

# Filters
nk ports --tcp
nk ports --udp
nk ports --listening
nk ports --established

# Advanced filtering
nk ports --state LISTEN,ESTAB --port 80,443 --process nginx --sort peer --limit 20
nk ports --pid 1234 --established
```

> Process information may require elevated privileges.

### Route Tracing

```bash
# Basic traceroute
nk trace example.com

# Add ISP / geolocation info
nk trace example.com --enrich
```

### MTU Discovery

```bash
# Automatically discover path MTU (binary search 68-9000)
nk mtu example.com

# Custom range
nk mtu vpn.example.com --min-size 1280 --max-size 1500
```

### MTR - Network Path Quality

`nk mtr` combines `ping` + `traceroute` to continuously monitor network path quality and collect metrics for each hop.

```bash
# Continuous monitoring (Ctrl+C to stop)
nk mtr example.com

# Limit the number of loops
nk mtr example.com -c 3

# Custom parameters
nk mtr example.com -m 20 -i 500 -t 5
# -m: maximum hops (default 30)
# -i: loop interval in milliseconds (default 1000)
# -t: timeout per probe in seconds (default 3)
```

### Network Traffic Monitoring (`top`)

`nk top` shows per-process network traffic in real time, similar to a bandwidth-oriented `top`.

```bash
# Live monitoring (updates every second)
sudo nk top

# Custom refresh interval / item count
sudo nk top -i 2 -n 10

# Single snapshot
sudo nk top --once

# Filter by interface
sudo nk top --iface eth0

# Sort by rate
sudo nk top --sort total|sent|received

# Total mode (no sudo required, interface-level traffic only)
nk top --total
```

### Packet Capture Monitoring (`watch`)

`nk watch` uses libpcap for live packet capture monitoring and can show aggregate statistics or live streams.

```bash
# Aggregate statistics (all interfaces, refresh every 5 seconds)
sudo nk watch

# Custom interval / limit
sudo nk watch --interval 10 --limit 30

# Single snapshot
sudo nk watch --once

# Specific interface
sudo nk watch --iface eth0

# Preset filters
sudo nk watch --preset synscan    # detect SYN scans
sudo nk watch --preset arpstorm   # ARP storms
sudo nk watch --preset mdns       # mDNS traffic
sudo nk watch --preset broadcast  # broadcast packets

# Custom BPF filter
sudo nk watch --filter "tcp port 80"
sudo nk watch --filter "host 192.168.1.100"

# Export pcap
sudo nk watch --iface eth0 --preset dns --pcap-out capture.pcap --duration 10
```

### mDNS / Bonjour Discovery

`nk mdns` discovers services broadcast via mDNS / Bonjour, which is common in IoT and office networks.

```bash
# Discover all services
nk mdns

# Specify a service type
nk mdns -s _http._tcp

# Resolve addresses / ports
nk mdns --resolve

# List all service types
nk mdns --list-types

# Custom timeout (default 5 seconds)
nk mdns -t 10
```

### WiFi Scanning

`nk wifi` shows current connection details and scans nearby networks.

```bash
# Current connection
nk wifi

# Scan (sorted by signal strength)
nk wifi --scan

# Filter by SSID / MAC / signal
nk wifi --scan --name "MyNetwork"
nk wifi --scan --mac "00:11:22"
nk wifi --scan --min-rssi -70

# Limit result count
nk wifi --scan --limit 10

# Specify interface
nk wifi --scan --iface wlan0
```

### Bluetooth Scanning

`nk bt` scans Bluetooth devices and is suitable for IoT / smart-home environments.

```bash
# Scan devices (10 seconds, sorted by signal)
nk bt --scan

# Custom timeout
nk bt --scan --timeout 15

# List paired / known devices
nk bt --list

# Device info (only works for paired devices)
nk bt --info AA:BB:CC:DD:EE:FF
```

### WHOIS Lookup

```bash
# Domain / IP WHOIS
nk whois example.com
nk whois 8.8.8.8

# Raw output
nk whois example.com --raw
```

### Speed Test

```bash
# Run speedtest (requires speedtest-cli)
nk speedtest

# Pass speedtest-cli arguments through
nk speedtest --simple
```

### Bandwidth Test (`iperf3`)

```bash
# Client mode: connect to an iperf3 server
nk iperf -c server.example.com

# Specify test duration
nk iperf -c server.example.com -t 30

# UDP test with bandwidth limit
nk iperf -c server.example.com -u -b 100M

# Reverse mode (server sends, client receives)
nk iperf -c server.example.com -R

# Multiple parallel streams
nk iperf -c server.example.com -P 4

# Custom port
nk iperf -c server.example.com -p 5201

# Server mode (run on the server side)
nk iperf -s
```

`nk iperf` automatically adds `-J` to obtain JSON output and parses it into structured results. All standard iperf3 parameters are supported.

### Network Diagnostics

```bash
# One-shot health check
nk diag

# Custom target
nk diag --target 1.1.1.1

# Skip selected tests
nk diag --no-public --no-http
```
