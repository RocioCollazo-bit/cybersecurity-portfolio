# 🌐 Network Traffic Analysis Lab

> Hands-on packet analysis lab simulating real-world network investigation using Wireshark and command-line tools to detect anomalies, investigate protocols, and identify suspicious activity.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Skills Demonstrated](#skills-demonstrated)
- [Lab Environment](#lab-environment)
- [Scenarios](#scenarios)
  - [Scenario 1 — Capturing Your First Packets](#scenario-1--capturing-your-first-packets)
  - [Scenario 2 — DNS Traffic Analysis](#scenario-2--dns-traffic-analysis)
  - [Scenario 3 — TCP/IP Handshake Investigation](#scenario-3--tcpip-handshake-investigation)
  - [Scenario 4 — HTTP Traffic Inspection](#scenario-4--http-traffic-inspection)
  - [Scenario 5 — Detecting Suspicious Activity](#scenario-5--detecting-suspicious-activity)
- [Wireshark Filter Reference](#wireshark-filter-reference)
- [Key Findings](#key-findings)
- [Lessons Learned](#lessons-learned)
- [How to Reproduce This Lab](#how-to-reproduce-this-lab)

---

## Overview

This lab simulates a network analyst's workflow: capturing live traffic, filtering by protocol, reconstructing conversations, and spotting anomalies that could indicate reconnaissance, data exfiltration, or misconfigured services.

All captures were done on a controlled local network. No external systems were targeted.

**Tools used:** Wireshark · tcpdump · tshark · nmap (for traffic generation)

---

## Skills Demonstrated

| Skill | Tools Used |
|---|---|
| Live packet capture | Wireshark, tcpdump |
| Display filter writing | Wireshark filter syntax |
| Protocol dissection | Wireshark protocol analyzer |
| DNS query/response analysis | Wireshark, tshark |
| TCP handshake reconstruction | Wireshark TCP stream follow |
| HTTP request/response inspection | Wireshark HTTP dissector |
| Suspicious traffic detection | Wireshark, custom filters |
| Command-line packet analysis | tcpdump, tshark |
| Writing capture reports | Manual documentation |

---

## Lab Environment

- **OS:** Ubuntu 22.04 LTS (Wireshark 4.x)
- **Network:** Controlled local network / loopback interface
- **Sample PCAP files:** Provided in `/captures/` folder (safe, pre-recorded traffic)
- **Alternative:** Use any of the public PCAP samples from [Wireshark's sample captures page](https://wiki.wireshark.org/SampleCaptures)

> ⚠️ All packet captures in this lab were performed on networks I own and control. Never capture traffic on networks without explicit permission.

---

## Scenarios

### Scenario 1 — Capturing Your First Packets

**Goal:** Understand how Wireshark captures traffic and what raw packets look like.

#### Using Wireshark (GUI)

```
1. Open Wireshark
2. Select your network interface (e.g., eth0, wlan0, or lo for loopback)
3. Click the blue shark fin button to start capture
4. Open a browser and visit http://example.com
5. Click the red square to stop capture
6. Save as: captures/first_capture.pcapng
```

#### Using tcpdump (command line)

```bash
# Capture 100 packets on any interface and save to file
sudo tcpdump -i any -c 100 -w captures/first_capture.pcap

# Capture only on eth0, show output in terminal
sudo tcpdump -i eth0 -v

# Capture and show packet contents in hex + ASCII
sudo tcpdump -i eth0 -XX -c 20
```

#### What you see in a raw packet

```
Frame 1: 74 bytes on wire
  Ethernet II
    Source: 00:0c:29:ab:cd:ef
    Destination: ff:ff:ff:ff:ff:ff
  Internet Protocol Version 4
    Source: 192.168.1.105
    Destination: 93.184.216.34
  Transmission Control Protocol
    Source Port: 54321
    Destination Port: 80
    Flags: SYN
```

**Key insight:** Every packet has layers — Ethernet → IP → TCP/UDP → Application data. Wireshark dissects all of them automatically.

---

### Scenario 2 — DNS Traffic Analysis

**Goal:** Understand how DNS queries and responses work, and spot unusual DNS behavior.

#### Wireshark filter

```
dns
```

#### What normal DNS looks like

```bash
# Generate DNS traffic to analyze
nslookup google.com
nslookup github.com
nslookup nonexistent-domain-xyz123.com   # This will produce a DNS error response
```

#### tcpdump — capture DNS only

```bash
# Capture only DNS traffic (port 53)
sudo tcpdump -i any port 53 -w captures/dns_traffic.pcap

# Read and display a saved DNS capture
sudo tcpdump -r captures/dns_traffic.pcap -v
```

#### tshark — extract DNS queries from a capture file

```bash
# List all DNS queries
tshark -r captures/dns_traffic.pcap -Y "dns.flags.response == 0" \
  -T fields -e frame.number -e ip.src -e dns.qry.name

# List all DNS responses with answer
tshark -r captures/dns_traffic.pcap -Y "dns.flags.response == 1" \
  -T fields -e frame.number -e ip.src -e dns.qry.name -e dns.a
```

#### Sample output — normal DNS exchange

```
Frame 10 | Query:    192.168.1.105 → 8.8.8.8  | google.com  (Type A)
Frame 11 | Response: 8.8.8.8 → 192.168.1.105  | google.com → 142.250.80.46

Frame 20 | Query:    192.168.1.105 → 8.8.8.8  | nonexistent-xyz.com (Type A)
Frame 21 | Response: 8.8.8.8 → 192.168.1.105  | NXDOMAIN (no such domain)
```

#### 🚩 Red flags in DNS traffic

| Anomaly | What it might mean |
|---|---|
| High volume of NXDOMAIN responses | Domain generation algorithm (DGA) malware |
| DNS queries to unusual ports (not 53) | DNS tunneling attempt |
| Extremely long domain names | Data exfiltration via DNS |
| Queries to unknown/private IPs | Rogue DNS server |
| Same domain queried hundreds of times | Beaconing malware |

---

### Scenario 3 — TCP/IP Handshake Investigation

**Goal:** Reconstruct TCP connections and understand the 3-way handshake.

#### Wireshark filters

```
# All TCP traffic
tcp

# Only SYN packets (new connections being initiated)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Only RST packets (connections being forcefully closed)
tcp.flags.reset == 1

# Specific port
tcp.port == 80

# Follow a specific TCP stream (right-click any packet → Follow → TCP Stream)
tcp.stream eq 0
```

#### The 3-way handshake — what to look for

```
Client (192.168.1.105)              Server (93.184.216.34)
        |                                   |
        |-------- SYN (Seq=0) ------------->|   Step 1: Client says "I want to connect"
        |                                   |
        |<------- SYN-ACK (Seq=0, Ack=1) --|   Step 2: Server says "OK, I'm ready"
        |                                   |
        |-------- ACK (Ack=1) ------------->|   Step 3: Client confirms
        |                                   |
        |======= DATA TRANSFER =============|   Connection established
        |                                   |
        |-------- FIN ----------------------|   Step 4: Client closes connection
        |<------- FIN-ACK ------------------|
        |-------- ACK ----------------------|
```

#### tcpdump — capture a TCP handshake

```bash
# Generate traffic: connect to a web server
curl http://example.com &

# Capture with TCP flag details
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn|tcp-ack|tcp-fin|tcp-rst) != 0' \
  -w captures/tcp_handshake.pcap

# Read and display flags clearly
sudo tcpdump -r captures/tcp_handshake.pcap -v \
  'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

#### 🚩 Red flags in TCP traffic

| Anomaly | What it might mean |
|---|---|
| SYN flood — many SYNs, no ACKs | Denial of Service (DoS) attack |
| Many RST packets from server | Port scan being rejected |
| SYN to many different ports rapidly | Port scanning activity |
| Half-open connections that never complete | Stealth/SYN scan (nmap -sS) |
| Retransmissions on every packet | Network instability or packet loss |

---

### Scenario 4 — HTTP Traffic Inspection

**Goal:** Inspect unencrypted HTTP traffic to see exactly what data is transmitted.

> ℹ️ This scenario uses plain HTTP (port 80) intentionally — to show why HTTPS matters. In production, always use HTTPS.

#### Wireshark filters

```
# All HTTP traffic
http

# Only HTTP GET requests
http.request.method == "GET"

# Only HTTP POST requests (form submissions, logins)
http.request.method == "POST"

# HTTP responses with error codes
http.response.code >= 400

# Search for a specific host
http.host contains "example"

# Filter by response content type
http.content_type contains "text/html"
```

#### Generate HTTP traffic to analyze

```bash
# Make HTTP requests (generates traffic to capture)
curl -v http://httpbin.org/get
curl -v -X POST http://httpbin.org/post -d "username=testuser&password=testpass"
curl -v http://httpbin.org/headers
```

#### tshark — extract HTTP data from capture

```bash
# Extract all HTTP request URIs
tshark -r captures/http_traffic.pcap -Y "http.request" \
  -T fields -e ip.src -e http.host -e http.request.uri -e http.request.method

# Extract HTTP response codes
tshark -r captures/http_traffic.pcap -Y "http.response" \
  -T fields -e ip.dst -e http.response.code -e http.content_length

# Export HTTP objects (files transferred over HTTP)
# In Wireshark: File → Export Objects → HTTP
```

#### Sample output — HTTP GET request visible in plain text

```
GET /login HTTP/1.1
Host: insecure-site.com
User-Agent: Mozilla/5.0 ...
Cookie: session=abc123def456

HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=newtoken789
```

#### 🚩 Red flags in HTTP traffic

| Anomaly | What it might mean |
|---|---|
| POST to `/login` over plain HTTP | Credentials sent in cleartext |
| Large POST body to unknown host | Data exfiltration |
| HTTP on unusual ports (not 80/8080) | Evasion technique |
| Encoded/obfuscated URIs | Web attack attempt |
| Requests to IP addresses (no hostname) | Malware C2 communication |
| User-Agent strings with "curl", "python" | Automated/scripted requests |

---

### Scenario 5 — Detecting Suspicious Activity

**Goal:** Apply everything learned to spot anomalies in a network capture.

#### The investigation

We received a PCAP file from a server that was behaving strangely. Let's analyze it.

```bash
# Step 1: Get a high-level summary of the capture
capinfos captures/suspicious_traffic.pcap

# Step 2: See what hosts are communicating
tshark -r captures/suspicious_traffic.pcap \
  -T fields -e ip.src -e ip.dst | sort | uniq -c | sort -rn | head 20

# Step 3: See what protocols are present
tshark -r captures/suspicious_traffic.pcap -q -z io,phs

# Step 4: Check for port scanning (many SYNs to different ports)
tshark -r captures/suspicious_traffic.pcap -Y "tcp.flags.syn==1 and tcp.flags.ack==0" \
  -T fields -e ip.src -e ip.dst -e tcp.dstport | sort | uniq -c | sort -rn

# Step 5: Look for large data transfers (possible exfiltration)
tshark -r captures/suspicious_traffic.pcap \
  -T fields -e ip.src -e ip.dst -e frame.len | \
  awk '{sum[$1" "$2]+=$3} END {for (k in sum) print sum[k], k}' | sort -rn | head 10

# Step 6: Check for DNS anomalies
tshark -r captures/suspicious_traffic.pcap -Y "dns" \
  -T fields -e ip.src -e dns.qry.name | \
  awk '{print length($2), $0}' | sort -rn | head 10
```

#### Investigation findings — what we found

```
Finding 1: PORT SCAN DETECTED
  Source IP: 192.168.1.200 sent SYN packets to 1,024 different ports
  on 192.168.1.105 within 3 seconds.
  → Likely: nmap port scan. Investigate host 192.168.1.200.

Finding 2: SUSPICIOUS DNS QUERIES
  Host 192.168.1.105 queried domains with names 60+ characters long.
  Pattern: random-looking subdomains of the same base domain.
  → Likely: DNS tunneling or DGA malware. Isolate host.

Finding 3: UNENCRYPTED CREDENTIALS
  Frame 847: POST to http://192.168.1.1/admin
  Body contains: username=admin&password=admin123
  → Critical: Admin credentials sent over plain HTTP.
```

#### Wireshark filters used in investigation

```
# Find port scan traffic
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Find long DNS names (possible tunneling)
dns.qry.name matches "[a-z0-9]{40,}"

# Find POST requests (possible credential theft)
http.request.method == "POST"

# Find large packets (possible exfiltration)
frame.len > 1400

# Find traffic to non-standard ports
not (tcp.port == 80 or tcp.port == 443 or tcp.port == 53 or tcp.port == 22)
```

---

## Wireshark Filter Reference

### Basic Protocol Filters

```
dns                     # All DNS traffic
http                    # All HTTP traffic
tcp                     # All TCP traffic
udp                     # All UDP traffic
icmp                    # Ping traffic
arp                     # ARP (address resolution)
tls                     # Encrypted traffic (HTTPS)
```

### IP Address Filters

```
ip.addr == 192.168.1.1          # Any packet from or to this IP
ip.src == 192.168.1.1           # Only FROM this IP
ip.dst == 192.168.1.1           # Only TO this IP
ip.addr == 192.168.1.0/24       # Entire subnet
not ip.addr == 192.168.1.1      # Exclude an IP
```

### Port Filters

```
tcp.port == 80                  # HTTP
tcp.port == 443                 # HTTPS
tcp.port == 22                  # SSH
tcp.dstport == 443              # Destination port only
tcp.srcport == 54321            # Source port only
```

### Combining Filters

```
ip.src == 192.168.1.1 and tcp.port == 80        # AND
dns or icmp                                      # OR
not arp                                          # NOT
http and ip.dst == 93.184.216.34                # Combined
```

### TCP Flag Filters

```
tcp.flags.syn == 1 and tcp.flags.ack == 0       # SYN only (new connections)
tcp.flags.reset == 1                             # RST (forced close)
tcp.flags.fin == 1                               # FIN (graceful close)
tcp.analysis.retransmission                      # Retransmissions
tcp.analysis.duplicate_ack                       # Duplicate ACKs
```

---

## Key Findings

| # | Finding | Severity | Protocol | Description |
|---|---|---|---|---|
| 1 | Port scan detected | 🔴 High | TCP | 1024 SYN packets in 3 seconds from single host |
| 2 | DNS tunneling suspected | 🔴 High | DNS | Anomalously long domain names, high query frequency |
| 3 | Cleartext credentials | 🔴 High | HTTP | Admin password visible in POST request body |
| 4 | Unencrypted admin panel | 🟡 Medium | HTTP | Admin interface accessible over HTTP (not HTTPS) |
| 5 | NXDOMAIN flood | 🟡 Medium | DNS | 200+ failed DNS lookups suggesting DGA malware |

---

## Lessons Learned

1. **Packet analysis is the ground truth.** Logs can be tampered with, but raw packets don't lie. If you want to know exactly what happened on a network, PCAP files are the answer.

2. **HTTP exposes everything.** Usernames, passwords, cookies, session tokens — all visible in plain text. This is why HTTPS is non-negotiable for anything sensitive.

3. **DNS is the most overlooked protocol.** Attackers love DNS because it's almost never blocked. Data exfiltration via DNS (DNS tunneling) is real, subtle, and hard to detect without packet-level visibility.

4. **Port scans have a clear signature.** A burst of SYN packets to sequential ports from a single source is unmistakable in Wireshark. Recognizing the pattern is half the battle.

5. **Baselines matter.** You can't spot anomalies without knowing what "normal" looks like. Building a baseline of your network's typical traffic is step one in any monitoring strategy.

6. **tshark is underrated.** The command-line version of Wireshark is incredibly powerful for scripted analysis, automation, and working on headless servers where you can't open a GUI.

---

## How to Reproduce This Lab

### Option A — Use the provided capture files

```bash
# Clone this repo
git clone https://github.com/YOUR_USERNAME/network-traffic-analysis
cd network-traffic-analysis

# Open any .pcap file in Wireshark
wireshark captures/dns_traffic.pcap
wireshark captures/http_traffic.pcap
wireshark captures/tcp_handshake.pcap

# Or analyze from command line
tshark -r captures/dns_traffic.pcap
```

### Option B — Generate your own traffic

```bash
# Install tools
sudo apt update
sudo apt install wireshark tshark tcpdump curl nmap

# Run the capture scripts
bash scripts/01_capture_dns.sh
bash scripts/02_capture_tcp.sh
bash scripts/03_capture_http.sh
bash scripts/04_investigate.sh
```

### Option C — Use public PCAP samples

Great sources for real-world packet captures to practice with:

- [Wireshark Sample Captures](https://wiki.wireshark.org/SampleCaptures)
- [Malware Traffic Analysis](https://www.malware-traffic-analysis.net/)
- [PCAP Files on GitHub](https://github.com/search?q=pcap+samples)

---

*This lab was built as part of my cybersecurity and network analysis portfolio. All traffic captures were generated in a controlled, isolated lab environment.*
