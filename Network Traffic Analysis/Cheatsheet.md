# Wireshark & tcpdump — Cheat Sheet

## Wireshark Display Filters

### By Protocol
```
dns           # DNS queries and responses
http          # Plain HTTP traffic
tcp           # All TCP connections
udp           # All UDP traffic
icmp          # Ping / ICMP
arp           # Address resolution
tls           # Encrypted traffic (HTTPS, etc.)
ftp           # File transfer
ssh           # Secure shell
smtp          # Email (outgoing)
```

### By IP Address
```
ip.addr == 192.168.1.1          # From OR to this IP
ip.src == 192.168.1.1           # Only FROM this IP
ip.dst == 192.168.1.1           # Only TO this IP
ip.addr == 192.168.1.0/24       # Entire subnet
not ip.addr == 192.168.1.1      # Exclude an IP
```

### By Port
```
tcp.port == 80                  # HTTP
tcp.port == 443                 # HTTPS
tcp.port == 22                  # SSH
tcp.port == 53                  # DNS (TCP)
udp.port == 53                  # DNS (UDP)
tcp.dstport == 443              # Destination port only
tcp.srcport == 54321            # Source port only
```

### TCP Flags
```
tcp.flags.syn == 1 and tcp.flags.ack == 0    # New connections (SYN)
tcp.flags.syn == 1 and tcp.flags.ack == 1    # Server accepts (SYN-ACK)
tcp.flags.reset == 1                          # Force-close (RST)
tcp.flags.fin == 1                            # Graceful close (FIN)
tcp.analysis.retransmission                   # Retransmitted packets
tcp.analysis.duplicate_ack                    # Duplicate ACKs
```

### HTTP Filters
```
http                                    # All HTTP
http.request                            # Requests only
http.response                           # Responses only
http.request.method == "GET"            # GET requests
http.request.method == "POST"           # POST requests
http.response.code == 200               # OK responses
http.response.code >= 400               # Error responses
http.host contains "google"             # Host header match
http.request.uri contains "login"       # URI contains word
```

### DNS Filters
```
dns                                     # All DNS
dns.flags.response == 0                 # Queries only
dns.flags.response == 1                 # Responses only
dns.flags.rcode == 0                    # Successful responses
dns.flags.rcode == 3                    # NXDOMAIN (domain not found)
dns.qry.name contains "google"          # Query for specific domain
dns.a == 8.8.8.8                        # DNS response with this IP
```

### Combining Filters
```
ip.src == 192.168.1.1 and tcp.port == 80        # AND
dns or icmp                                      # OR
not arp                                          # NOT
http and ip.dst == 93.184.216.34                # IP + protocol
tcp.port == 80 or tcp.port == 443               # Multiple ports
```

### Packet Size
```
frame.len > 1400          # Large packets
frame.len < 100           # Small packets
frame.len == 1514         # Exact size (max Ethernet frame)
```

---

## tcpdump Quick Reference

### Basic Capture
```bash
# Capture on specific interface
sudo tcpdump -i eth0

# Capture and save to file
sudo tcpdump -i any -w output.pcap

# Capture limited number of packets
sudo tcpdump -i any -c 100 -w output.pcap

# Read a saved file
sudo tcpdump -r output.pcap

# Verbose output
sudo tcpdump -i eth0 -v
sudo tcpdump -i eth0 -vvv         # Even more verbose

# Show packet contents (hex + ASCII)
sudo tcpdump -i eth0 -XX
```

### Filter by Protocol
```bash
sudo tcpdump -i any tcp
sudo tcpdump -i any udp
sudo tcpdump -i any icmp
```

### Filter by Port
```bash
sudo tcpdump -i any port 80
sudo tcpdump -i any port 53
sudo tcpdump -i any 'port 80 or port 443'
sudo tcpdump -i any 'not port 22'        # Exclude SSH noise
```

### Filter by IP
```bash
sudo tcpdump -i any host 192.168.1.1
sudo tcpdump -i any src 192.168.1.1
sudo tcpdump -i any dst 192.168.1.1
sudo tcpdump -i any net 192.168.1.0/24
```

### Filter by TCP Flags
```bash
# SYN packets only
sudo tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0'

# RST packets only
sudo tcpdump -i any 'tcp[tcpflags] & tcp-rst != 0'

# SYN + ACK (handshake response)
sudo tcpdump -i any 'tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)'
```

### Useful Combinations
```bash
# HTTP traffic on any interface, save to file
sudo tcpdump -i any port 80 -w http.pcap

# DNS traffic, verbose
sudo tcpdump -i any port 53 -v

# Everything except SSH (avoids noise when connected via SSH)
sudo tcpdump -i eth0 not port 22 -w traffic.pcap

# Capture between two specific hosts
sudo tcpdump -i any 'host 192.168.1.1 and host 192.168.1.2'
```

---

## tshark Quick Reference

```bash
# Read a pcap and display summary
tshark -r file.pcap

# Filter like Wireshark
tshark -r file.pcap -Y "http.request"

# Extract specific fields
tshark -r file.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport

# Protocol statistics
tshark -r file.pcap -q -z io,phs

# Top talkers
tshark -r file.pcap -T fields -e ip.src | sort | uniq -c | sort -rn

# Export HTTP objects
tshark -r file.pcap --export-objects http,/tmp/http-objects/

# Live capture (same as tcpdump)
sudo tshark -i eth0 -Y "dns"
```

---

## Common Investigation Workflow

```
1. capinfos file.pcap              → Summary: duration, packet count, size
2. tshark -q -z io,phs             → What protocols are in there?
3. Top talkers (ip.src count)      → Who is generating the most traffic?
4. Filter suspicious protocol      → Drill into DNS, HTTP, TCP flags
5. Follow TCP/UDP Stream           → Reconstruct full conversations
6. Export objects                  → Pull out transferred files
7. Document findings               → What was normal? What was anomalous?
```
