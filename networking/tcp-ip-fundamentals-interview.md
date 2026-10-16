# TCP/IP Fundamentals - SRE Interview Guide

**Time to master:** 30 minutes  
**Difficulty:** ⭐⭐ (Easy-Medium)  
**Interview frequency:** High (foundational, often asked conceptually)

---

## ⚡ Quick Summary

TCP/IP is a **layered architecture** where data travels down through layers (each adding headers) when sending, and back up when receiving. SREs need to:
- **Understand the layers** (Link → IP → TCP → Application)
- **Know MAC vs IP addressing** (hardware vs software)
- **Understand encapsulation** (headers nested like envelopes)
- **Know MTU and fragmentation** (why packets have size limits)
- **Know ports and services** (how multiple services run on one IP)

---

## 🎯 Key Concepts to Know

### 1. **The TCP/IP Layer Model**

```
┌─────────────────────────────────────────┐
│  APPLICATION LAYER                      │  HTTP, DNS, SSH, SMTP
│  (Services, Protocols)                  │
├─────────────────────────────────────────┤
│  TRANSPORT LAYER                        │  TCP, UDP
│  (Reliable/Unreliable Delivery)         │
├─────────────────────────────────────────┤
│  INTERNET LAYER                         │  IP (IPv4, IPv6), ARP
│  (Routing, Addressing)                  │
├─────────────────────────────────────────┤
│  LINK LAYER                             │  Ethernet, WiFi, PPP
│  (Hardware, Frames, MAC Addresses)      │
├─────────────────────────────────────────┤
│  PHYSICAL LAYER                         │  Cables, Signals
│  (Actual Wires)                         │
└─────────────────────────────────────────┘
```

**Each layer talks to the layer above and below it:**
- Application sends data to Transport
- Transport adds headers, sends to Internet
- Internet adds headers, sends to Link
- Link adds headers, sends to Physical
- Physical wire carries the bits

**Each layer has different names for the same data:**
- Link layer: **Frame**
- Internet layer: **Packet**
- Transport layer: **Segment** (TCP) or **Datagram** (UDP)
- Application layer: **Data** or **Message**

---

### 2. **MAC Addressing (Link Layer)**

**Purpose:** Identify devices on the same physical network

```
Format: 48 bits = 6 bytes
Example: 00:50:8D:9A:3B:DF
         ^^^^^^           ← Manufacturer (OUI - Organizationally Unique Identifier)
                ^^^^^^   ← Device serial number

Scope: Local network only (doesn't route beyond network segment)
Used by: ARP (maps IP to MAC), Ethernet switches, DHCP servers
```

**Key facts:**
- Permanent, but can be spoofed
- Only devices on same network segment can use it
- Switches use MAC to forward frames
- ARP protocol converts IP → MAC

---

### 3. **IP Addressing (Internet Layer)**

**Purpose:** Identify devices globally across the internet

```
IPv4 Format: 32 bits = 4 octets
Example: 192.168.1.5
         ↑    ↑    ↑ ↑
         |    |    | └─ Host ID
         |    |    └──── Subnet
         |    └───────── Network
         └──────────────── Class

Scope: Global (routes across internet)
Used by: Routers for forwarding, applications for connectivity
```

**Key facts:**
- Identifies network interfaces, not machines (one computer = multiple IPs)
- Hierarchical (network part + host part)
- Routable (unlike MAC)
- Private ranges exist (10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12)

---

### 4. **Packet Encapsulation**

When you send data, each layer adds its own header (like nested envelopes):

```
SENDING:
Application: "Hello Server"
              ↓ (Add TCP header)
Transport:   [TCP Header | Hello Server]
              ↓ (Add IP header)
Internet:    [IP Header | TCP Header | Hello Server]
              ↓ (Add Ethernet header)
Link:        [Ethernet Header | IP Header | TCP Header | Hello Server]
              ↓ (Add FCS checksum)
Physical:    Bits on the wire →

RECEIVING (reversed):
Physical wire → Bits arrive
              ↓ (Remove Ethernet header, check FCS)
Link:        [IP Header | TCP Header | Hello Server]
              ↓ (Remove IP header, check routing)
Internet:    [TCP Header | Hello Server]
              ↓ (Remove TCP header, deliver to port)
Transport:   [Hello Server]
              ↓ (Deliver to application)
Application: "Hello Server" ✓
```

**Why this matters:**
- Each layer adds overhead (headers)
- Total packet size limited by MTU
- Fragmentation happens if packet too large

---

### 5. **MTU (Maximum Transfer Unit)**

**Definition:** Maximum size of data that can fit in one frame

```
Standard values:
├─ Ethernet: 1,500 bytes
├─ Wireless (WiFi): 1,500 bytes
├─ PPP (dial-up): 296-1,500 bytes
├─ Tunneled (VPN): ~1,480 bytes (smaller due to tunnel header)
└─ Jumbo frames: 9,000 bytes (special networks)

Example with Ethernet 1,500 byte MTU:
┌─────────────────────────────────┐
│ Ethernet Header: 14 bytes       │
├─────────────────────────────────┤
│ IP Header: 20 bytes             │
├─────────────────────────────────┤
│ TCP Header: 20 bytes            │
├─────────────────────────────────┤
│ Application Data: ~1,446 bytes  │ ← What's left
├─────────────────────────────────┤
│ Checksum: 4 bytes               │
└─────────────────────────────────┘
  Total: 1,500 bytes
```

**If data is too large:**
- TCP/IP automatically fragments (splits into smaller packets)
- Router on the path reassembles
- Fragmentation = performance hit

**Path MTU Discovery:**
- TCP checks: "What's the smallest MTU on the path to this host?"
- Sets packet size accordingly
- Avoids fragmentation

---

### 6. **Ports and Services**

**Definition:** 16-bit number (0-65,535) that identifies a specific service/process

```
Well-known ports (0-1,023):
├─ HTTP: 80
├─ HTTPS: 443
├─ SSH: 22
├─ SMTP: 25
├─ DNS: 53
└─ Others: /etc/services file

Registered ports (1,024-49,151):
├─ Used by applications (databases, cache servers, etc.)
└─ Can be used by non-root users

Dynamic/Private ports (49,152-65,535):
├─ Temporary ports assigned by OS
└─ Used for outgoing connections
```

**Key fact:** IP address identifies the machine, port identifies the service

```
192.168.1.5:80    = HTTP server on machine 192.168.1.5
192.168.1.5:443   = HTTPS server on same machine (different service!)
192.168.1.5:22    = SSH server on same machine (different service!)

All three use same IP, different ports
```

---

### 7. **Address Types**

| Type | Definition | Example | Use Case |
|------|-----------|---------|----------|
| **Unicast** | One sender to one specific receiver | 192.168.1.5 | Normal communication |
| **Broadcast** | One sender to all hosts on subnet | 192.168.1.255 | DHCP requests, ARP queries |
| **Multicast** | One sender to group of receivers | 224.0.0.1 | Video streaming, notifications |
| **Anycast** | Packets go to nearest of several servers | Root DNS servers | Load balancing |

**SRE relevance:**
- **Unicast:** Standard, day-to-day
- **Broadcast:** DHCP and ARP (you'll see in troubleshooting)
- **Multicast:** Rare, but understand it exists
- **Anycast:** Root DNS uses it, understand the concept

---

## ❓ Typical Interview Questions

### **Q1: "Explain what happens when you ping google.com"**

**Your answer (showing layer understanding):**

```
1. Application Layer:
   "Your ping app sends an ICMP Echo Request"

2. Internet Layer:
   "It adds an IP header with:
    - Source IP: 192.168.1.5 (your machine)
    - Destination IP: 8.8.8.8 (google.com after DNS)
   
3. Link Layer:
   "It needs to get to the gateway (192.168.1.1)
   Uses ARP to get the gateway's MAC address: aa:bb:cc:dd:ee:ff
   Adds Ethernet header with:
   - Source MAC: your MAC
   - Destination MAC: gateway's MAC
   
4. Physical Layer:
   "Packet travels on the wire to the gateway
   Gateway sees it, looks at IP header, routes to next hop
   Eventually reaches 8.8.8.8
   
5. Response comes back (reversed):
   8.8.8.8 → ... → your machine
   Ping app receives ICMP Echo Reply
   Prints: '64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=20.1 ms'
"
```

**Why this works:** Shows you understand all layers and data flow

---

### **Q2: "Why can't I reach host 10.0.0.50 from my machine on 192.168.1.5?"**

**Your answer (systematic thinking):**

```
1. First check: Can I route there at all?
   ping 10.0.0.50
   
2. If no response:
   "This IP is on a different network (different subnet).
   I need a route to it.
   
   Check routing table:
   route -n
   
   If no route exists, packets will be dropped at my gateway."

3. If route exists but still fails:
   "The gateway has a route, but what about the intermediate routers?
   Check with:
   traceroute 10.0.0.50
   
   Tells me which hop fails."

4. If it reaches the destination but no response:
   "Network is routed correctly, but destination not answering.
   Either:
   - Host is down
   - Firewall is blocking
   - ARP not resolving (if on same network)
   
   Check if on same network:
   route -n (look at netmask)
   
   If same network, host should respond to ARP anyway.
   So it's probably down or firewalled."
```

**Why this works:** Systematic, layer-by-layer troubleshooting

---

### **Q3: "A server is listening on TCP port 8080, but clients can't connect. What could be wrong?"**

**Your answer:**

```
Issues at different layers:

1. Application Layer (port 8080):
   "Service might not be listening on 8080
   Check: ss -tln | grep 8080
   
   Or listening but crashing:
   Check logs, is it actually running?"

2. Transport Layer (TCP):
   "Check if port is open:
   ss -tln
   
   Or service is listening on wrong IP:
   Is it listening on 0.0.0.0:8080 (all interfaces)?
   Or 127.0.0.1:8080 (localhost only)?"

3. Internet Layer (routing):
   "Can you reach the host at all?
   ping <host_ip>
   
   Is there a route to it?
   route -n"

4. Link Layer (network):
   "Firewall blocking?
   iptables -L (on host)
   or check security group (if cloud)"

"I'd test each layer systematically:
- Service running? systemctl status
- Listening on correct port? ss -tln
- Can reach host? ping
- Route exists? route -n
- Firewall allows? iptables/security group"
```

**Why this works:** Shows layer awareness and systematic debugging

---

### **Q4: "What's the difference between MAC and IP addressing? When do I need each?"**

**Your answer:**

```
MAC Address:
- Hardware address (burned into NIC)
- 6 bytes: aa:bb:cc:dd:ee:ff
- Local network only (doesn't route)
- Used by: switches, ARP

IP Address:
- Software address (assigned by OS)
- 4 bytes (IPv4) or 16 bytes (IPv6)
- Global scope (routable across internet)
- Used by: routers, applications

When you send data:
1. IP routing determines path to destination
2. At each network segment, ARP maps IP → MAC
3. Switches forward using MAC
4. Routers forward using IP

Example - sending data to 8.8.8.8 from 192.168.1.5:
- My machine: "Destination is 8.8.8.8 (IP routing)"
- Check: Is it on my network? NO
- Send to gateway: 192.168.1.1
- ARP: What's the MAC of 192.168.1.1?
- Get: aa:bb:cc:dd:ee:ff
- Frame: Destination MAC = aa:bb:cc:dd:ee:ff, 
         Destination IP = 8.8.8.8
- Switch forwards to gateway (using MAC)
- Gateway forwards to next hop (using IP)
"
```

**Why this works:** Shows you understand the relationship between layers

---

## 🚨 Common Mistakes

❌ **"MAC addresses route across the internet"**  
✅ MAC only works on local network. Routers use IP.

❌ **"IP addresses identify computers"**  
✅ IP addresses identify network interfaces. One computer can have multiple IPs.

❌ **"I don't need to understand MTU for SRE"**  
✅ MTU issues cause mysterious slow connections. Understanding fragmentation helps.

❌ **"TCP and IP are the same thing"**  
✅ TCP is Transport layer, IP is Internet layer. Different layers, different jobs.

❌ **"All packets on a link have the same size"**  
✅ Different packets can be different sizes (up to MTU).

---

## 🛠️ Essential Commands (Quick Reference)

```bash
# View IP configuration
ip addr show                    # IP addresses on all interfaces
ip route show                   # Routing table

# View MAC addresses
ip link show                    # MAC addresses and interface status
arp -a                          # ARP table (IP ↔ MAC mappings)

# View network connections by port
ss -tln                         # TCP listening sockets
ss -tan                         # All TCP connections
lsof -i :8080                   # What's using port 8080

# Check MTU
ip link show | grep mtu         # MTU of each interface
ping -M do google.com           # Test path MTU (don't fragment)

# DNS to IP (verify routing target)
dig google.com +short           # Get IP address
```

---

## 🎬 Real-World Scenario

**Situation:** "Application is unreachable. Works from same network but fails from different network."

**Diagnosis (using layer knowledge):**

```
1. Works on same network:
   "ARP resolves, MAC address found, packets reach app"

2. Fails from different network:
   "ARP still works (different ARP table)
    But IP routing fails"

Root cause: No route between networks

Check:
- From failing machine: ping <app_ip>
- Check routing: route -n
- Is there a gateway/router between networks?
- Is firewall blocking traffic between networks?

Layer analysis:
- Link layer: ✓ ARP works on both networks
- Internet layer: ✗ No route between networks
- Transport/App: Never reach this far
```

---

## 📊 Interview Tips

✅ **Show layer thinking:** "At the link layer..., at the IP layer..."  
✅ **Connect layers:** Show how they work together  
✅ **Use terminology:** Frame, packet, segment, datagram  
✅ **Ask clarifying questions:** "Is this on the same network?"  
✅ **Reference commands:** Back up your reasoning with tools  

---

## 🔗 Related Files

- [tcp.md](./tcp.md) - TCP detailed reference
- [dns.md](./dns.md) - DNS resolution (uses IP)
- [curl-tools.md](./curl-tools.md) - Tools that work on these layers
- [networking-interview-guide.md](./networking-interview-guide.md) - Master networking guide

---

## 📚 Key Takeaways

✓ **TCP/IP is layered:** Each layer adds headers, handles one job  
✓ **MAC is local:** Only devices on same network segment  
✓ **IP is global:** Routes across the internet  
✓ **Ports identify services:** Not just the machine  
✓ **MTU limits packet size:** Fragmentation is a performance hit  
✓ **Encapsulation:** Each layer nests inside the layer below  
✓ **Layers work together:** Routing (IP) at each hop, MAC for local delivery  

---

**Next:** Review [networking-interview-guide.md](./networking-interview-guide.md) for practical scenarios.
