
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
