# 1. What is an IP Address? (for Cybersecurity Beginners)

### 1. The Postcard Analogy

Imagine sending a postcard:

- You need a **destination address** (so the mailman knows where to deliver).
- You also need a **return address** (so the receiver knows who sent it).

On the internet, **IP addresses are those addresses**. They tell computers where to send data packets and where they came from.
### 2. Two Generations of IPs

- **IPv4**: The classic, 32-bit system. Example: `192.168.1.10`
    - Think of this like an old city with **limited house numbers** (about 4.3 billion).

- **IPv6**: The modern, 128-bit system. Example: `2001:db8:abcd::1`
    - Like a new megacity with **practically unlimited house numbers**.


💡 Fun fact: If IPv4 was a bucket of water, IPv6 would be all the oceans combined!

### 3. Cybersecurity Lens 🔍

Why IPs matter in security:
- Attackers hide behind **fake IPs (spoofing)**.
- Security teams use IPs in **firewalls, intrusion detection, and attribution**.
- Misconfigured IPs can expose systems to the wrong networks.
### 4. Quick Self-Test

- Look up your current IP address (Google: “what is my IP”).
- Is it **public** (visible on the internet) or **private** (internal to your Wi-Fi)?
- Bonus: If you’re on Wi-Fi, check your local IP too (`ipconfig` on Windows, `ip addr` on Linux). Do you see two different ones? Why might that be? (Hint: NAT 👀 — we’ll cover that later.)

### 5. Key Takeaway

An IP is your device’s “street address” on a network. Without it, your data has nowhere to go. From a cybersecurity perspective, **IP = identity + traceability (but only partially trustworthy)**.

# 2. IPv4 vs IPv6 — The Two Generations of Internet Addresses

### 1. The Big Picture

There are two main “versions” of IP addresses in use today:
- **IPv4** → the older, widely used system (since the 1980s).
- **IPv6** → the newer, designed to replace IPv4 (launched in the 1990s, still rolling out).

Why two? Because the world ran out of IPv4 addresses. But IPv6 isn’t just “bigger numbers” — it changes some security and networking fundamentals too.

### 2. How They Look

- **IPv4**: 32 bits → 4 decimal numbers (`0–255`), separated by dots.  
    Example: `203.0.113.7`
- **IPv6**: 128 bits → 8 groups of hexadecimal numbers, separated by colons.  
    Example: `2001:db8:abcd:12::34`

📝 IPv6 shorthand rules:

- Drop leading zeros in each group.
- Collapse one run of consecutive zeros with `::`.
### 3. Address Space (Numbers Available)

- IPv4: ~4.3 billion addresses total (and we’ve used almost all of them).
- IPv6: ~340 undecillion (that’s a 3.4 × 10³⁸). Enough to give every grain of sand on Earth billions of addresses.

💡 Thought experiment: If IPv4 were a pizza with 8 slices, IPv6 is the **entire Milky Way galaxy of pizzas**.

### 4. Why Security People Care

**IPv4 Security Concerns**

- **Easy to scan**: Attackers can sweep the whole IPv4 internet (`nmap 0.0.0.0/0`) in hours or days.
- **NAT (Network Address Translation)** is common. That means multiple devices hide behind one public IPv4, making attribution messy.

**IPv6 Security Shifts**

- **Too big to scan**: You can’t brute-force search 2¹²⁸ addresses. Attackers rely on:
    - DNS lookups
    - Host leaks (misconfigured systems broadcasting details)
    - Guessable patterns (admins often pick “easy” IPv6 host IDs)
- **New attack surface**: IPv6 uses **Neighbor Discovery** (instead of ARP) and **extension headers**, which come with their own spoofing/evasion risks.
- **Dual stack**: Many networks run IPv4 _and_ IPv6. If you secure only IPv4, attackers may slip in via IPv6 unnoticed.

### 5. Quick Self-Test 

1. Run this command on your machine:
    - Windows: `ipconfig`
    - Linux/Mac: `ip addr`

2. Do you see an **IPv6 address** starting with `fe80::` or `2001::`?
3. Question: If your firewall rules only mention IPv4, what could an attacker do with that IPv6 address?

(Hint: They might bypass your defenses!)

### 6. Cybersecurity Takeaways

- IPv4 is familiar, small, and everywhere — but limited and often shared.
- IPv6 is massive, more private (when configured right), but adds **new protocols and pitfalls**.
- As a defender, **never ignore IPv6** — it’s probably already running in your network, whether you use it or not.

# 3. Special & Reserved IP Ranges (IPv4 & IPv6)

Not all IP addresses are “normal house addresses.” Some are **reserved neighborhoods** with special purposes. As a cybersecurity learner, recognizing them is critical — because attackers love to spoof them, and defenders must filter/log them carefully.

### 1. IPv4 Special Ranges

- **Private IPs (RFC1918)**
    - `10.0.0.0/8` (10.0.0.0 – 10.255.255.255)
    - `172.16.0.0/12` (172.16.0.0 – 172.31.255.255)
    - `192.168.0.0/16` (192.168.0.0 – 192.168.255.255)  
        Used inside LANs, Wi-Fi, corporate networks. Never routable on the internet.

- **Loopback**
    - `127.0.0.0/8` (commonly `127.0.0.1`)  
        Always refers to “this machine.” If you see it in network traffic, something fishy is going on.
        
- **Link-local (APIPA)**
    
    - `169.254.0.0/16`  
        Auto-assigned when no DHCP is found. Only works inside the local LAN segment.
        
- **Multicast**
    
    - `224.0.0.0/4`  
        👉 One-to-many traffic (e.g., video streaming, routing protocols).
        
- **Documentation/Test ranges**
    
    - `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`  
        👉 Used in textbooks, labs, and tutorials. Should never appear in real traffic.
        

---

### 2. IPv6 Special Ranges

- **Global Unicast (`2000::/3`)**  
    👉 Equivalent of “public IPv4.” These are routable on the internet.
    
- **ULA (Unique Local Addresses)**
    
    - `fc00::/7` (commonly `fdxx::`)  
        👉 Like private IPv4. Not meant for global internet.
        
- **Link-local (`fe80::/10`)**  
    👉 Every IPv6 device auto-generates one. Required for local communication.
    
- **Loopback (`::1`)**  
    👉 The IPv6 version of 127.0.0.1.
    
- **Multicast (`ff00::/8`)**  
    👉 IPv6 leans heavily on multicast (e.g., neighbor discovery, routing).
    
- **Documentation/Test (`2001:db8::/32`)**  
    👉 Appears in labs, books, and online tutorials.
    

---

### 3. Cybersecurity Relevance

- 🔎 **For defenders**: If you see traffic with a **private, loopback, or docs range coming from the internet**, that’s **spoofed traffic** — usually malicious.
    
- 🔒 **For attackers**: Spoofing private or loopback IPs can trick poorly configured firewalls or monitoring systems.
    
- 📡 **In logs**: Don’t panic if you see private ranges inside your LAN — but be very concerned if they appear at your **internet edge**.
    
- 🛑 **Loopback in the wild** = red flag. If you capture `127.0.0.1` or `::1` on a network tap, someone’s trying to evade detection.
    

---

### 4. Quick Self-Test

1. Your home Wi-Fi probably uses one of these private ranges.
    
    - Run `ipconfig` (Windows) or `ip addr` (Linux/Mac).
        
    - Which range is it using? 192.168.x.x? 10.x.x.x?
        
2. If your ISP gave your router an address like `10.x.x.x`, what does that mean?  
    _(Hint: That’s Carrier-Grade NAT, where you share a public IP with many customers.)_
    

---

### 5. Key Takeaway

👉 Knowing **special IP ranges** is like knowing reserved seats in a theater:

- Some are **local only**, some are **testing only**, some are **broadcast/multicast only**.
    
- Cybersecurity work requires instantly recognizing these to spot **spoofing, leaks, or misconfigurations**.
    

---

✨ Next up (Topic 4): **Subnetting Basics — Network bits vs Host bits (CIDR)**.  
This is where we roll up our sleeves and dive into the math — but I’ll make it visual and fun.

---

Do you want me to **jump straight into CIDR subnetting with binary & practice**, or first do a **gentle intro (what “network vs host bits” means with drawings/analogies)** before we crunch numbers?



# IPv4 vs IPv6 (and how they look)

- **IPv4**: 32 bits → 4 numbers `0–255`, e.g. `203.0.113.7`.
- **IPv6**: 128 bits → 8 groups of hex, e.g. `2001:db8:abcd:12::34`.  
    Shortcuts: leading zeros dropped, one `::` for a single run of zeros.

Security relevance:
- More IPv6 space → scanning the whole internet like we do with IPv4 isn’t feasible. Attackers lean on **DNS**, **host discovery leaks**, and **guessable addressing** instead.
- IPv6 has different **neighbor discovery** and **extension headers** that matter for defenses (more below).
# Network bits vs host bits (CIDR)

Each IP belongs to a subnet: **prefix/length** like `10.0.1.25/24`.
- `/24` → first 24 bits are the **network** (here `10.0.1.x`), last 8 bits are **host**s.
- Fewer prefix bits = bigger network. Quick mental map:
    - `/8` ≈ 16.7M hosts
    - `/16` ≈ 65,536 hosts
    - `/24` ≈ 256 hosts
- Same idea in IPv6, just… bigger. Typical LANs use `/64`.

Security relevance:
- Segmentation starts here. Smaller subnets limit lateral movement and blast radius.
- Mis-sized masks cause weird reachability and can expose hosts unintentionally.
# Special & private ranges that are seen a lot

**IPv4**

- **Private (RFC1918[^2]):** `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
- **Loopback:** `127.0.0.0/8` (usually `127.0.0.1`)
- **Link-local (APIPA[^3]):** `169.254.0.0/16` (no DHCP[^4])
- **Multicast:** `224.0.0.0/4`
- **Docs/test:** `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`

**IPv6**

- **Global unicast:** `2000::/3` (public internet)
- **ULA (private-ish):** `fc00::/7` (often shown `fdxx:`)
- **Link-local:** `fe80::/10`
- **Loopback:** `::1`
- **Multicast:** `ff00::/8`
- **Docs/test:** `2001:db8::/32`

Security relevance:
- Don’t blocklist these “just because”; they appear in **tunnels**, **logs**, and **lateral movement**.
- **Loopback** traffic shouldn’t hit the wire. If you see it on a span[^5] port, something’s off.
# How devices get an IP (and how attackers abuse that)

**IPv4 DHCP (DORA)**: Discover → Offer → Request → Ack.  
**IPv6**: SLAAC (router announcements) and/or DHCPv6.

Security angles:
- **Rogue DHCP** can push bad gateways/DNS. Mitigations: **DHCP snooping**, **RA Guard** (for IPv6).
- **DHCP starvation** can exhaust leases, causing DoS.
- **SLAAC privacy**: rotate interface IDs to avoid tracking; without it, EUI-64 may leak NIC MAC patterns.

# Name resolution matters (DNS)

Users type names, networks route to IPs. That translation process of hostnames to ip addresses is a juicy target.
- Attacks: **DNS poisoning**, **malicious resolvers**, **typo-squatting**.
- Defenses/observability: **DNSSEC**, **DoH/DoT**, **split-DNS**, and **logging**.  
    DNS logs plus IP flow logs = great for incident timelines.
# ARP & Neighbor Discovery (L2 ↔ L3 glue)

- **IPv4 ARP**: “Who has 10.0.1.25? Tell 10.0.1.1.”
- **IPv6 ND**: neighbor solicitation/advertisement, router advertisements.

Security angles:
- **ARP spoofing/poisoning** → man-in-the-middle. Mitigations: **Dynamic ARP Inspection**, static ARP for critical gear.
- **RA spoofing** (IPv6) can reroute traffic. Mitigation: Use **RA Guard** on switchports.
# NAT, PAT, and what they _do not_ do

- **NAT/PAT (NAPT)** maps many private IPs to one public IP with different source **ports**.
- **Port forwarding** pokes holes inbound to internal hosts.

Security truth:
- NAT is **not** a security control. It obscures, it doesn’t enforce. The actual guard is your **stateful firewall**.
- **CGNAT** at ISPs means multiple customers share a public IP—weakens IP-based attribution.
# IP + ports = where the action happens

- A “socket” = `{source IP:port → dest IP:port}` over **TCP** or **UDP**.
- **Well-known ports** (0–1023) host core services (22 SSH, 80 HTTP, 443 HTTPS, 53 DNS…).
- **Ephemeral ports** are picked by the OS for client connections.

Security angles:
- Allow-listing by **port** is blunt. Apps tunnel in **443** now; add **TLS inspection**, **app-layer controls**, and **behavior**.
- **UDP** is stateless → easier for **reflection/amplification** (spoofed-source DDoS).

# ICMP (don’t just block it)

- Used for **ping**, **traceroute**, and **Path MTU Discovery**.
- Blindly blocking ICMP can break PMTUD → mysterious “it’s slow/broken” tickets.
- Rate-limit and filter specific types instead of a blanket deny.
# Inside the IP header (what defenders look at)

**IPv4 fields:** version, **header length**, **DSCP/ECN**, **total length**, **ID**, **flags/fragment offset**, **TTL**, **protocol**, **checksum**, source/dest.

- **TTL**: helps with **traceroute** and fingerprinting OS defaults.
- **Fragmentation**: reassembly bugs & **IDS evasion** (overlaps, tiny fragments). Prefer to **drop or reassemble** at the edge; set **DF** with proper PMTUD.
- **Options** are rare; seeing them can be suspicious.

**IPv6 fields:** version, **traffic class**, **flow label**, **payload length**, **next header**, **hop limit**, plus **extension headers**.

- Long chains of extension headers or odd ones (e.g., routing headers) are commonly filtered due to evasion risk.

# Attribution & privacy (what an IP can/can’t tell you)

- A public source IP often maps to: **one device**, **a NAT’d household**, **a whole mobile cell**, or **a VPN/proxy/Tor exit**.
- ISPs rotate addresses; orgs use **proxies** and **NAT**; clouds reuse pools.  
    → Treat IP as **weak identity**. Correlate with **auth logs**, **user-agents**, **JA3/TLS**, **cookies**, **EDR**.

# Common attack/abuse patterns tied to IPs

- **Source IP spoofing** (mostly with UDP) → reflection/amplification (DNS, NTP, SSDP, memcached). Mitigation: **ingress anti-spoofing** (BCP-38) and **egress filters** on your network.
- **Recon/scanning**: ping sweeps, TCP SYN scans, service/version probes. Legal note: only scan what you own/manage or have explicit permission to test.
- **Fragmentation/TTL games**: IDS/IPS evasion and weird paths.
- **IPv6-specific**: RA spoofing, ND cache mischief, extension header abuse.

# Practical defensive moves with IPs

1. **Segmentation by subnet/VLAN**
    - Split users/servers/OT/IoT. Apply least privilege between subnets with **stateful firewalls**.

2. **Anti-spoofing everywhere**
    - Edge: drop private/bogon sources on internet-facing interfaces. Inside: filter “impossible” sources on each VLAN.

3. **Logging strategy**
    - Keep **DHCP leases**, **NAT translations**, **proxy** and **VPN** logs with time sync (NTP). They’re gold for investigations.

4. **Rate-limit & filter** at edges
    - ICMP (surgically), UDP to known amplifiers, abnormal IPv6 extension chains, and tiny/overlapping fragments.

5. **IPv6-ready security**
    - Enable **RA Guard**, **DHCPv6 Guard**, and IPv6 rules that mirror your IPv4 policy. Don’t ignore IPv6—it’ll still be on.

6. **GeoIP & reputation lists**
    - Useful signal, not gospel. Pair with behavior analytics and authentication controls.

# CLI cheat-sheet (safe, routine commands)

> Replace targets with hosts you own or have permission to test.

```bash
# See your addresses
ip addr            # Linux
ip route
ifconfig           # older *nix
ipconfig /all      # Windows
Get-NetIPAddress   # PowerShell

# Basic reachability & path
ping 8.8.8.8
traceroute 8.8.8.8         # Linux/macOS
tracert 8.8.8.8            # Windows

# DNS → IP
nslookup example.com
dig A example.com +short

# Who owns an IP (high-level)
whois 203.0.113.7

# Quick host discovery in your lab subnet (ARP-based, low-noise)
arp -a
nmap -sn 192.168.1.0/24    # discovery only; requires permission
```

# Mini lab exercises

1. **Map your LAN**
    - Find your IP/mask and default gateway.
    - Identify your `/24` (or `/23`) and list 3 live hosts with `ping` or `nmap -sn`.

2. **Trace the internet path**
    - `traceroute` to a public DNS (e.g., 1.1.1.1). Note the hop count and where private vs public IPs appear.

3. **DHCP story**
    - Release/renew your lease; capture with Wireshark. Label Discover/Offer/Request/Ack.
    - Spot the **server IP**, **your new IP**, **lease time**.

4. **ICMP filtering test**
    
    - `ping` a site that drops ICMP. Compare traceroute that uses UDP vs ICMP; note differences.

# Quick FAQ (common gotchas)

- **“NAT keeps me safe, right?”**  
    Not by itself. It obscures; your **firewall policy** and **patching** keep you safe.
- **“Why does the website still know me after my IP changed?”**  
    Cookies, browser fingerprinting, logged-in sessions, TLS client hints—IP is only one signal.
- **“Should we block all ICMP?”**  
    No. Keep essential types for PMTUD and diagnostics; rate-limit instead.
- **“Do we really need IPv6 security rules?”**  
    Yes. Hosts enable IPv6 by default. Unfiltered IPv6 is a sneaky backdoor.

# A tiny glossary

- **CIDR**: modern subnet notation (`/24`) replacing old A/B/C classes.
- **Default gateway**: next hop for traffic leaving your subnet.
- **Stateful firewall**: tracks connections so return traffic is allowed automatically.
- **BCP-38**: best practice to drop spoofed source IPs at the edge.
- **SLAAC**: IPv6 auto-addressing from router announcements.

# 1) Binary fundamentals — the foundation (don’t skip this)

- IPv4 = 32 bits. We split into 4 octets of 8 bits: `[octet1].[octet2].[octet3].[octet4]`.
- Each bit is worth `2^position` from right to left within the octet: `128 64 32 16 8 4 2 1`.
- Example: decimal `192` = binary `11000000` because `128 + 64 = 192`. `10` = `00001010` (8 + 2).

Why binary matters: subnet masks, network boundaries, AND operations — all are bitwise. You will convert many addresses mentally if you understand place values.

Small practice (do 1 minute): write binary of 10, 64, 128, 192.  

### Network address vs Broadcast address 

- **Network address** = the identifier for the subnet. It’s the IP with all **host bits = 0**. Routers use it to identify the route/prefix.
- **Broadcast address** = the special IP inside the subnet with all **host bits = 1**. It's used to send a packet to **all hosts** in that subnet (Layer-2 broadcast / ARP / legacy IPv4 broadcasts).

How to compute (concept)

Given `IP / prefix`:
1. Convert IP and mask to 32-bit binary.
2. **Network address** = `IP & MASK` (bitwise AND) → host bits zeroed.
3. **Broadcast address** = `NETWORK | WILDCARD` where `WILDCARD = ~MASK` (host bits set to 1).

Example: `192.168.2.100/27`

- `/27` → mask `255.255.255.224` → last octet mask `1110 0000` (host bits = 5).
- IP last octet `100` = `0110 0100`.
- Network last octet = `0110 0100 AND 1110 0000 = 0110 0000` → `96`.
- Broadcast last octet = `0110 0000 OR 0001 1111 = 0111 1111` → `127`.

So:
- **Network:** `192.168.2.96/27`
- **Broadcast:** `192.168.2.127`
- **Usable hosts:** `192.168.2.97` → `192.168.2.126`

Usability & special cases

- Traditional usable host range = `network + 1` through `broadcast − 1`.
- **/31** (2 addresses) — RFC 3021: used for point-to-point links; there is **no broadcast** and both addresses are usable.
- **/32** — single host, no host bits — used to refer to one address.
- IPv6 has **no broadcast**; it uses **multicast** (e.g., `ff02::1` for all-nodes).
### How broadcasts are used (IPv4)

- **ARP requests** use layer-2 broadcast to resolve MAC addresses (`who-has 192.168.2.105` → broadcast).
- **DHCP discovery** uses broadcast (client often uses `0.0.0.0` → `255.255.255.255` or DHCP server broadcast).
- **Directed broadcast**: packet addressed to the broadcast address of a remote subnet (e.g., `192.168.1.255`) — routers commonly block directed broadcasts because of amplification (smurf attacks).

Quick reminder

- **Network address** = subnet identifier (all host bits `0`). Example: `192.168.2.96/27`.  
    Not normally assigned to a host.
- **Broadcast address** = address in the subnet with all host bits `1`. Example: `192.168.2.127`.  
    Used to reach all hosts in the subnet at once.

### When the **network address** is used (and why)

1. **Routing / route entries**
    - Routers and OS routing tables store prefixes as `network/prefix` (e.g., `192.168.2.96/27`) — that network address is the canonical key for the route.
    - Example: `ip route add 192.168.2.96/27 via 10.0.0.1` — tells the kernel “to reach anything in that network, send to this next hop.”
    - **Why:** router lookups match prefixes to decide next-hop — the network address is how we identify the prefix.

2. **Network configuration & documentation**
    - Admins use the network address to name subnets (like VLAN 20 → `192.168.2.96/27`). DHCP pools, ACLs, and monitoring rules are typically specified with the network prefix.
    - **Why:** it’s the single, canonical identifier for that subnet.

3. **Route summarization (supernetting)**
    - Aggregating many subnets into a single larger `network/prefix` for smaller routing tables (ISPs do this).
    - **Why:** scale: reduces number of routes advertised.

4. **Access Control Lists / Firewall rules**
    - Use the `network/prefix` to permit/deny ranges. Example: `allow 10.0.0.0/8` or `deny 192.168.2.96/27`.
    - **Why:** compactly express rules for many hosts.

5. **Network calculations and tools**
    - Subnet calculators, IPAM entries, `ipcalc` outputs use the network address to derive ranges.

6. **Special cases**
    - The **network address is never** used as a host destination in normal IPv4 practice (it identifies the subnet). If you try to assign it to a host, routing/misconfiguration problems follow.
    - `/31` and `/32` are special (RFC 3021) — with `/31` both addresses are usable for point-to-point links, so “network” semantics differ.

### When the **broadcast address** is used (and why)

1. **ARP (Address Resolution Protocol)** — the classic example
    - A host needing a MAC for `192.168.2.105` sends an ARP request as an L2 broadcast (`ff:ff:ff:ff:ff:ff`) with L3 target `192.168.2.105`. All hosts on that LAN see the frame; the owner replies.
    - **Why:** ARP is link-local and uses broadcast because the sender doesn’t yet know the MAC of the destination.

2. **DHCP Discovery / DHCP servers**
    - Clients with no IP often broadcast DHCPDISCOVER to find servers. DHCP responses sometimes use broadcast if the client cannot yet receive unicast.
    - **Why:** client lacks an IP/ARP entry; broadcast reaches all DHCP servers on the LAN.

3. **Wake-on-LAN (WOL)** and other “wake everyone” or discovery messages
    - WOL often uses a directed or limited broadcast to reach network interface controllers that listen for magic packets.
    - **Why:** need to reach NICs irrespective of current IP stack state.

4. **Local service discovery and legacy broadcasts**
    - NetBIOS name service, some discovery protocols, multicast-to-broadcast fallbacks, and simple custom discovery probes use broadcasts to find peers.
    - **Why:** convenient “everyone on this LAN” reachability without prior configuration.

5. **Directed broadcast** (sending an IP packet to `network.broadcast` of a _remote_ subnet)
    - Example: from 10.0.0.5 route to `192.168.2.127` which is the broadcast of the `192.168.2.96/27` subnet. Routers can be configured to forward this as a L2 broadcast into the remote subnet.
    - **Why (legitimate uses):** remote administration, WOL across subnets.
    - **Why (dangerous):** historically used as amplification (e.g., **Smurf attack**) — many routers now block or rate-limit directed broadcasts.

6. **Local network scanning / discovery**
    - Admins sometimes ping the broadcast to elicit replies from many hosts (useful for discovery). Many modern hosts or OSes ignore/respond differently for security reasons.

7. **Router behavior**
    - **Routers typically do not forward link-layer broadcasts.** Directed IP broadcasts are a special case and are often disabled by default for security.

### L2 vs L3 broadcasts — important distinction

- **L2 broadcast** = Ethernet frame `ff:ff:ff:ff:ff:ff`. Seen only inside a broadcast domain (switch + VLAN). ARP uses this.
- **L3 (IP) broadcast** includes:
    - **Limited broadcast:** `255.255.255.255` — local-subnet only, never forwarded by routers.
    - **Directed broadcast:** e.g., `192.168.2.127` for `192.168.2.96/27` — can be forwarded if router permits.

- **Why this matters:** L2 broadcasts do not cross routers; directed broadcasts are controlled and can be a security risk.

### Practical examples (commands / config)

- **Routing table entry:** `ip route show` will display `192.168.2.96/27 via 10.0.0.1 dev eth0` — that `network/prefix` is how kernel matches destinations.
- **ARP:** `sudo tcpdump -n -e arp` will show ARP requests as L2 broadcasts (sample output shows `ff:ff:ff:ff:ff:ff`).
- **Ping broadcast (discovery, often blocked):** `ping -b 192.168.2.127` (Linux). Many systems ignore broadcast pings by default.

### Security implications & mitigations (very important)

- **Amplification / reflection attacks**: directed broadcasts can turn a single spoofed packet into many replies (historical smurf).  
    **Mitigate:** disable directed broadcast forwarding on routers, ingress filtering (uRPF), block broadcast-addressed traffic at the edge.
- **Broadcast storms**: buggy devices or malicious floods of ARP/broadcast traffic can saturate a LAN.  
    **Mitigate:** VLAN segmentation, switch storm-control, rate-limiting, and isolation.
- **ARP spoofing / poisoning**: attackers exploit ARP’s trust model to intercept traffic.  
    **Mitigate:** Dynamic ARP Inspection, static ARP entries for critical endpoints, 802.1X, port security.
- **Discovery abuse**: broad scan via broadcast/ping can be used for reconnaissance.  
	**Mitigate:** host firewalls rejecting broadcast requests, disable unnecessary broadcast-based services.    
- **Logging / forensics**: broadcast floods can make logs noisy — include thread/host info and filter by network to find real incidents.

### Rules of thumb

- Use broadcasts for _local_ link discovery, ARP, DHCP, and WOL — not for general cross-subnet commands.
- Prefer **multicast** for scaled “many receivers” scenarios (it’s routable with control and less noisy).
- Keep broadcast domains small with VLANs to limit blast radius.
- Treat directed broadcasts as risky — keep them disabled unless you need them for a legit use-case (and secure them).

In short, 

- **Network address** = used to _identify_ the subnet — central to routing, ACLs, and IPAM. Not a host destination.
- **Broadcast address** = used to _reach every host_ on that subnet for ARP, DHCP, discovery, WOL, etc. Routers usually don’t forward link-layer broadcasts; directed broadcasts are special and potentially dangerous.
- **Security:** minimize broadcast domains, disable/limit directed broadcast forwarding, use ARP protections and rate-limits.

### Commands & quick code

Python (std lib) to compute:

```python
import ipaddress
net = ipaddress.ip_network("192.168.2.100/27", strict=False)
print(net.network_address, net.broadcast_address)
# => 192.168.2.96 192.168.2.127
```

On Linux, `ipcalc` or `ip route` / `ip addr` can help (if installed).

# 2) CIDR and subnet masks — the language

- CIDR `/n` = first `n` bits are network. Mask = `n` ones followed by `32-n` zeros.
- **CIDR** — _Classless Inter-Domain Routing_ — is the modern way to represent IP address ranges by specifying an IP **prefix** and a **prefix length**. Instead of the old classful A/B/C buckets, CIDR lets you say “this many bits are the network” and the rest are host bits. It’s compact, flexible, and essential for subnetting, routing, and any network work.
- Dotted-decimal mask derived by summing octet bit values. Example:
    - `/8` = `11111111 . 00000000 . 00000000 . 00000000` = `255.0.0.0`
    - `/16` = `255.255.0.0`
    - `/24` = `255.255.255.0`
    - `/26` = `255.255.255.192` because last octet `11000000` = `128+64 = 192`.

- Usable hosts = `2^(32 - n) - 2` (subtract network & broadcast). Exceptions: point-to-point `/31`, `/32`.

Why subtract 2? Network address (all host bits 0) and broadcast (all host bits 1) cannot be assigned to hosts on “traditional” LANs. (/31 for p2p and /32 for a single-host route are exceptions explained below.)

# 3) Three practical ways to calculate network & broadcast (use whichever fits)

### Method A — Bitwise AND (precise, always works)

1. Convert IP and mask to 32-bit binary.
2. Network = IP AND Mask (bitwise).
3. Broadcast = Network OR (inverse of mask).  
    Example: `192.168.10.130/26`

- Mask `/26` = `255.255.255.192` → last octet mask bits `11000000`.
- Convert last octet 130 = `10000010`.
- AND => `10000000` (128) → network last octet = `.128`. So network `192.168.10.128`. Broadcast = network + 63 = `.191`.

Use when crossing octet boundaries — programmatic and robust.

### Method B — Increment / Block size method (fast mental)

1. Find host bits = `32 - n`. Block size = `2^(host bits)` = addresses per subnet.
2. Find the octet that changes (the octet where the mask is not 255).
3. Subnet increments = `256 / block_size_in_that_octet` OR simpler: list boundaries: 0, block, 2×block...
4. Round the IP down to the nearest boundary to get network.

Example: `/26` → host bits 6 → block size 64. Boundaries in last octet: 0,64,128,192. IP `192.168.10.130` → falls in `128–191` block → network `.128`, broadcast `.191`.

Great for quick human work.

### Method C — Decimal math across octets (carry method)

When the boundary is not in the last octet:

- Work with the 32-bit number or treat higher octets as an aggregate and carry remainders.
- Example: `10.0.5.200/20`. `/20` = mask `255.255.240.0`. The third octet is partially network (`1111 0000`) so blocks of size 16 in the third octet: networks at `0,16,32,...,240`. `5` falls in `0` block → network `10.0.0.0/20`.

This is useful for larger networks and VLSM.

# 4) Useful cheat table (memorize these common subnet sizes)

|Prefix|Mask|Total addr|Usable hosts|Block size (last changing octet)|
|---|--:|--:|--:|---|
|/24|255.255.255.0|256|254|1 (256 per /24)|
|/25|255.255.255.128|128|126|128|
|/26|255.255.255.192|64|62|64|
|/27|255.255.255.224|32|30|32|
|/28|255.255.255.240|16|14|16|
|/29|255.255.255.248|8|6|8|
|/30|255.255.255.252|4|2|4|
|/31|255.255.255.254|2|2*|2|
|/32|255.255.255.255|1|1*|1|

* `/31` used for point-to-point links: RFC 3021 allows assigning both addresses (no network/broadcast). `/32` is single host (often used in routing to represent loopback or host routes).

Memorize the powers-of-two sequence: 256,128,64,32,16,8,4,2,1 — it will speed you up.
# 5) Special cases & gotchas

- **/31**: Useful for P2P links to save IPs (no broadcast). Modern routers support RFC3021.
- **/32**: Host route, loopback addresses (not usable as "normal" host on LAN without proxying).
- **Zero subnets**: Historically some networks forbade using the all-zero subnet (`10.0.0.0/8` split into `10.0.0.0/9` etc.) — modern practice allows them.
- **Directed Broadcasts**: `192.168.1.255` for `192.168.1.0/24`. Older routers could forward directed broadcasts and cause amplification (smurf attacks). Many networks disable directed-broadcast forwarding for security.
- **Fragmentation**: IP-level fragmentation can break some tools. MTU and DF flag matter.

# 6) VLSM (Variable Length Subnet Mask) & Summarization

- VLSM: carve a big network into subnets of **different sizes** to save addresses. Workflow: assign largest subnets first (greedy algorithm).
- Example: from `10.0.0.0/24`, need subnets for 100 hosts, 50 hosts, 10 hosts:
    - Use /25 for 100 hosts (126 usable), /26 for 50 hosts (62 usable), /28 for 10 hosts (14 usable)
- Route summarization (supernetting): combine contiguous subnets into a single route advertised in routing protocols to shrink routing tables. Supernetting works when network addresses are aligned on the summarization boundary.

Important: When doing VLSM, always list required host counts and pick the smallest prefix that fits each host requirement.

# 7) Practical commands & lab tools

- Linux:
    - `ip addr show` (view addresses)
    - `ip route` (routing table)
    - `ipcalc 192.168.1.130/26` (if installed) or `sipcalc` (more features)
    - `nmap -sL 192.168.1.0/24` (list hosts)

- Windows:
    - `ipconfig /all
    - `route print`

- Cross-platform:
    - `whois`, `dig`, `nslookup` (useful adjuncts for service discovery)

- Web/CMD calculators: `ipcalc` (online) or smartphone CIDR calculators — useful when learning.

# 8) Mental shortcuts & mnemonics (speed tricks)

- To find block size of `/n`: `block = 2^(8 - (n % 8))` for the octet where the mask stops. Or simpler: `2^(32-n)` is total addresses for subnet.
- To round down to network quickly in the octet: subtract (IP octet % block_size).
- Mnemonic for reading masks: think in octets of `128,64,32,16,8,4,2,1` — the mask octet is the sum of highest values.
- For incremental boundaries: memorise 0,128,64,32,16,8,4,2,1 steps as breakpoints depending on prefix.

# 9) Worked example (slightly larger, multi-octet)

Given `172.16.45.201/21`:

- `/21` → host bits = 11 → addresses per subnet = `2^11 = 2048`.
- `/21` mask = `255.255.248.0`. Note third octet mask is `248` → block size in 3rd octet = 8 (because 256 / (addresses_in_third_octet) => but simpler: mask 248 = 11111000 → block of 8 in third octet).
- Third octet `45` → nearest boundary: multiples of 8 → `40, 48...` → `45` falls in `40` block.
- Network = `172.16.40.0/21`. Broadcast = `172.16.47.255`. Usable hosts `172.16.40.1 - 172.16.47.254`.

This is precisely where decimal math across octets saves time.

# 10) Labs & practice checklist (do these in order)

1. Convert these decimals to binary: 10, 128, 191, 224. (1 minute)
2. Using the increment method: find network/broadcast/hosts for:
    - `192.168.100.70/26` (you already saw a similar one)
    - `10.0.12.130/25`
    - `172.16.200.150/20`

3. Use `ipcalc`/`sipcalc` on your machine to verify the above.
4. VLSM exercise: Given `10.10.0.0/20`, split into `8 equal subnets` — try this _now_. (Don’t tell me the final list yet if you want coaching; if you want direct check, paste your answers and I’ll verify.)
5. Build a tiny lab: three VMs on VirtualBox with addresses in the same /26, ping between them, then change one’s mask to /25 and observe reachability changes.

# 11) Red/Blue Practical Notes — why this matters in ops

Red team:

- Subnet calculations let you plan scans without tipping off defenders (scan only relevant subnets, use /32 pings to avoid noisy broadcast storms).
- Understanding NAT & private ranges helps craft C2 flows and bypassing egress filters.
- IPv4 arithmetic helps design stealthy beacon intervals and C2 subnets.

Blue team:

- IP math is essential for creating ACLs and firewall rules that block or allow precise ranges.
- Summarization knowledge prevents over-broad ACLs that break services.
- Knowing directed-broadcast and /31 behavior helps detect unusual traffic patterns (e.g., P2P links with /31 used by attackers to hide).

# 12) Common mistakes made

- Counting hosts incorrectly (forgetting -2 for network & broadcast).
- Relying solely on dotted decimal masks without thinking in binary — leads to errors when masks cross octets.
- Not planning VLSM from largest to smallest — wastes address space.
- Confusing block size (addresses) with usable hosts.

[^1]: RFC 1

[^2]: RFC 1918 is an Internet Engineering Task Force (IETF) standard that designates specific IPv4 address ranges for use in private networks

[^3]: APIPA, or Automatic Private IP Addressing, is a feature in operating systems like Windows that allows a device to automatically assign itself an IP address when a [DHCP server](https://www.google.com/search?client=firefox-b-d&sca_esv=356e0bcf11d0cff3&sxsrf=AE3TifNQ0GdZ8JzL6y18dGtIbnXgC6C5nA%3A1758263911667&q=DHCP+server&sa=X&ved=2ahUKEwir94C_m-SPAxXUTGwGHf6LLk4QxccNegQIMhAC&mstk=AUtExfC5Mo21bHo8GKIYSuqyfxEEaceIfVrs97ZZGOJl5cxShCsIvrz5OGb6lOSxVexOazdTAZI2AwBaJkl_uQi9qIMSixZIDTxThbYx_XQuBtEDlkJxRdtKbOOisMzOK4HHc64w21h1BpygleQaC-tvoE-jE4XNUHmhfGLdh_ZVBHB_DJS4TrI1dwwFij6PQhzyCze8so0UmLhL8c0gEqndQIfNplfKClmVz4e4jY4O1Rw5lUcgg8ouilocjDoD_a8d-ks-_vVk8Sdke-XpFCXAWZyL&csui=3) (a network server that assigns IP addresses) is unavailable. This feature uses IP addresses in the reserved [169.254.0.0/16](https://www.google.com/search?client=firefox-b-d&sca_esv=356e0bcf11d0cff3&sxsrf=AE3TifNQ0GdZ8JzL6y18dGtIbnXgC6C5nA%3A1758263911667&q=169.254.0.0%2F16&sa=X&ved=2ahUKEwir94C_m-SPAxXUTGwGHf6LLk4QxccNegQINBAB&mstk=AUtExfC5Mo21bHo8GKIYSuqyfxEEaceIfVrs97ZZGOJl5cxShCsIvrz5OGb6lOSxVexOazdTAZI2AwBaJkl_uQi9qIMSixZIDTxThbYx_XQuBtEDlkJxRdtKbOOisMzOK4HHc64w21h1BpygleQaC-tvoE-jE4XNUHmhfGLdh_ZVBHB_DJS4TrI1dwwFij6PQhzyCze8so0UmLhL8c0gEqndQIfNplfKClmVz4e4jY4O1Rw5lUcgg8ouilocjDoD_a8d-ks-_vVk8Sdke-XpFCXAWZyL&csui=3) range, enabling communication between devices on the same local network but not with devices on other networks or the internet.

[^4]: DHCP, or [Dynamic Host Configuration Protocol](https://www.google.com/search?client=firefox-b-d&sca_esv=356e0bcf11d0cff3&sxsrf=AE3TifN7Wrdgz0dEL7CPsE91lnhyAMYRUA%3A1758263971045&q=Dynamic+Host+Configuration+Protocol&sa=X&ved=2ahUKEwjFoqnbm-SPAxXLS3ADHX6oHHEQxccNegQIIRAB&mstk=AUtExfAf0Zn7PLVNwuLY7WaU3tGs1yMCNQYvIyjnGMxgMlCvRPwg-4wmf9-DeWMSdWWrWJd-zUlwWzNoPk6170V2gifpx7jE2deaLBE1QSy1qBJHRb9j1SXCY2oJoQjo0uJrs2z02z-102TKNFHK8gc1M3FzxFe8SK5XpTAgfYFNH6Nan4Q4lPNQnsXkhduo4HqY5UbqQRvg6gEFQNQUtm_dPYJOc62m2Jr-AKG4Ue5Ty8klxTjnpzGX2bdYWMYbTefQiITNkfTTaXqyWrenrDdy_NTy&csui=3), is a network protocol that automatically assigns IP addresses and other configuration information to devices on a network, such as computers, smartphones, and smart devices.

[^5]: A SPAN port, or [Switched Port Analyzer](https://www.google.com/search?client=firefox-b-d&sca_esv=356e0bcf11d0cff3&sxsrf=AE3TifMX0zFOnDguaQvgiZPHOteAhutQvw%3A1758264183482&q=Switched+Port+Analyzer&sa=X&ved=2ahUKEwj5tM_AnOSPAxV-X2wGHTGDNwwQxccNegQIFxAB&mstk=AUtExfDeGo_zoyu3z2ps80V3MKkm9iaBjxKllWk8VS6OxChdNfrn40YM8eaq_kw23DzFjdbrnoL3d1p9EgxPyZB_Isy9CAj354kxSlN2cf95UJvwjrYeCMmQPYsdWABoNZoYKV8sLjJ4OL2es5sJqB9y7IVbHZA5La8tuErrEZ1-_TORsicZpOZHUWJW4R0InNZvLJUa6ajsfkaIN9IEpkAtVnzhJ6YKQtOuvmQWAc0LRYJDhXHHDF8BAzy6NwK5CUcca3_bvvrYmFPm_IzgZppZRRt5&csui=3), is a software feature on network switches that copies and sends network traffic from one or more source ports to a designated destination port.
