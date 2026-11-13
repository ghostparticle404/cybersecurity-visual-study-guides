![Visual guide comparing IP addresses, MAC addresses, and ports, including local and routed network traffic](../00-visual-guides/01.05.IP%2C%20MAC%20%26%20Port%20Addressing%20Guide.png)

# 05 — IP Addresses, MAC Addresses & Ports

**Cybersecurity Visual Study Guides · 01 — Networking Fundamentals**  
**Estimated time:** 25–30 minutes, including practice

A device can have an IP address, a network interface can have a MAC address, and an application can listen on a port. Why do we need all three? This lesson follows a packet from a laptop to a web service, first on the same local network and then across a router, to show what each kind of addressing actually does.

## Why this matters

Imagine reviewing a packet capture that shows a laptop connecting to a web server. You see source and destination IP addresses, Ethernet addresses, and TCP port numbers. Which field tells you *where the packet is going across networks*? Which one tells you *where the current Ethernet frame is going on this link*? Which helps identify the *application conversation*?

Confusing those answers can lead to incorrect troubleshooting and incident-response conclusions. Knowing how the fields fit together helps you interpret captures, firewall rules, connection logs, and suspicious traffic.

## Learning objectives

By the end, you should be able to:

1. **Distinguish** IP addresses, MAC addresses, and transport-layer ports by their roles and scope.
2. **Explain** how ARP in IPv4 and Neighbor Discovery in IPv6 help determine a local next-hop link-layer address.
3. **Trace** a connection to a host on the same subnet and to a host reached through a router.
4. **Interpret** source and destination IPs, MACs, and TCP/UDP ports in a simple packet-capture scenario.
5. **Identify** what a socket endpoint represents and explain why a port alone does not uniquely identify a network conversation.

## The big picture, in plain English

These three identifiers answer different questions:

| Identifier | The question it helps answer | Scope |
|---|---|---|
| **IP address** | Which network-layer destination should this packet reach? | Used for delivery across IP networks |
| **MAC address** | Which interface is the next recipient on this local link? | Used on MAC-addressed links such as Ethernet and Wi-Fi |
| **Port number** | Which transport-layer endpoint or service should receive this TCP/UDP traffic? | Interpreted by the destination host's transport stack |

They are not interchangeable. A packet can keep its destination IP address while its Ethernet destination MAC address changes as it travels through routers. And the same host can support multiple services on different ports.

> **Important distinction:** Not every network link uses Ethernet-style MAC addresses, and not every application uses TCP or UDP ports. This lesson focuses on common IPv4/IPv6 networks carrying TCP or UDP over Ethernet/Wi-Fi.

## A memorable story: street address, name badge, office extension

You're delivering a document to an office building. First, you need the **street address** to find the building. When you arrive, you check the **name badge** of the person handling deliveries at that location. Finally, you call the correct **office extension** so the document reaches the intended department.

Now imagine that the office is in another city. A courier takes the document to a distribution center, then another courier handles the next leg. The final destination stays the same, but the person carrying the document changes at each handoff.

That is the useful mental picture:

- **IP address:** the street address that guides delivery across networks.
- **MAC address:** the name badge of the interface receiving the frame on the current link.
- **Port:** the office extension for the intended transport-layer endpoint.
- **Router:** a distribution center that chooses the next leg.

### Where the analogy stops working

An IP address is not necessarily a permanent physical location: it may be private, temporary, reassigned, or translated by Network Address Translation (NAT). A MAC address identifies an interface on a link, not necessarily a unique person or even a permanently unique device; addresses can be randomized or changed. A port is a numeric transport-layer identifier, not literally a phone extension or a guarantee that a particular application is running. And routers forward packets according to routing information, not human courier instructions.

## Translate the story into technical reality

| Story element | Real technical concept | Why it matters |
|---|---|---|
| Street address | Destination IP address | Guides network-layer delivery |
| Name badge | Next-hop destination MAC address on a MAC-addressed link | Directs the frame to the next interface on that link |
| Office extension | TCP or UDP destination port | Helps deliver traffic to the correct endpoint |
| Courier's return details | Source IP address and source port | Identify where replies should be directed, subject to routing and translation |
| Distribution center | Router | Forwards IP packets onto another link |
| Checking who receives deliveries | ARP or IPv6 Neighbor Discovery | Resolves the next-hop IP address to a link-layer address where needed |

Let's leave the building metaphor behind and follow real network traffic.

## How it actually works

### 1. IP addresses: network-layer delivery

An **Internet Protocol (IP)** address is used for network-layer addressing and routing. IPv4 addresses are 32 bits long, often written like `192.168.1.25`. IPv6 addresses are 128 bits long, often written like `2001:db8::25` (the `2001:db8::/32` range is reserved for documentation).

A device decides whether a destination is reachable on a directly connected network or whether it needs to send the packet to a next-hop router, based on its routing table and interface configuration. IP addresses identify interfaces or assigned network-layer endpoints in context; one device may have several IP addresses.

### 2. MAC addresses: delivery on the current link

A **Media Access Control (MAC)** address is a link-layer identifier used by technologies such as Ethernet and Wi-Fi. A common Ethernet MAC address is 48 bits, written like `02:11:22:33:44:55`.

On Ethernet, a frame includes a **source MAC** and a **destination MAC**. The destination MAC normally identifies the next receiving interface on that link—not necessarily the final IP destination. MAC addresses are not reliable proof of device identity: they can be locally administered, randomized, or spoofed.

### 3. Ports: transport-layer endpoints

**TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)** each use 16-bit source and destination port fields. A port number ranges from `0` to `65535`.

Common destination ports include:

| Port | Typical protocol or service | Caution |
|---|---|---|
| TCP 80 | HTTP | HTTP can use other ports |
| TCP 443 | HTTPS | Encrypted web traffic can use other ports; HTTP/3 commonly uses UDP 443 |
| TCP 22 | SSH | SSH servers can listen elsewhere |
| UDP/TCP 53 | DNS | DNS uses both transport protocols |

A **socket endpoint** is commonly described by a transport protocol, IP address, and port—for example, `TCP 192.168.1.25:51514`. A particular TCP connection is commonly distinguished by its source IP, source port, destination IP, and destination port, together with the transport protocol. A port number by itself does not uniquely identify a conversation.

### 4. ARP and Neighbor Discovery: finding the next hop

Before sending an Ethernet frame, a host may need to learn the link-layer address associated with the next-hop IP address.

- **ARP (Address Resolution Protocol)** resolves IPv4 addresses to link-layer addresses on an appropriate local network. A host commonly broadcasts an ARP request and receives a reply with the relevant MAC address.
- **IPv6 Neighbor Discovery (ND)** uses **ICMPv6 (Internet Control Message Protocol version 6)** messages, including Neighbor Solicitation and Neighbor Advertisement, to discover neighbors' link-layer addresses and perform other functions. It is more than a one-for-one replacement for ARP.

Hosts generally consult existing neighbor/cache entries before initiating resolution. ARP and ND are not used to discover the remote server's MAC address across the entire Internet.

### 5. Trace A: a connection on the same local network

Assume these illustrative addresses:

- **Laptop:** IP `192.168.1.25`, MAC `02:11:22:33:44:55`
- **Local server:** IP `192.168.1.50`, MAC `02:AA:BB:CC:DD:EE`
- **Service:** HTTPS on TCP port `443`
- **Laptop's temporary source port:** `51514`

The laptop wants to open a TCP connection to `192.168.1.50:443`.

1. The laptop's routing decision says the server is on a directly reachable local network.
2. If the server's MAC address is not already in its neighbor cache, the laptop uses **ARP** to resolve `192.168.1.50`.
3. It sends an Ethernet frame addressed to the **server's MAC**.
4. Inside the frame is an IP packet addressed to **`192.168.1.50`**.
5. Inside the IP packet is a TCP segment addressed to **destination port `443`**, with source port `51514`.
6. The server's TCP stack processes the segment and, if appropriate, delivers the connection to the listening service.

```text
SAME LOCAL NETWORK

Laptop 192.168.1.25                         Server 192.168.1.50
MAC 02:11:22:33:44:55                      MAC 02:AA:BB:CC:DD:EE
       |                                            ^
       +-------------- Ethernet frame -------------+

Ethernet:  02:11:22:33:44:55 -> 02:AA:BB:CC:DD:EE
IPv4:      192.168.1.25     -> 192.168.1.50
TCP:       51514            -> 443
```

The frame's destination MAC and the packet's destination IP both correspond to the local server, but they serve different layers.

### 6. Trace B: a connection across a router

Now the laptop connects to a remote example server at `203.0.113.10:443`. Assume its default gateway is `192.168.1.1` with MAC `02:66:77:88:99:AA`.

1. The laptop's routing table selects the **default gateway** as its next hop.
2. The laptop resolves the gateway's local MAC address using ARP, if necessary.
3. The laptop sends an Ethernet frame **to the router's MAC**, while the IP packet's destination remains **`203.0.113.10`**.
4. The router removes the incoming link-layer framing, examines the IP destination, selects an outgoing route, and sends the packet in a new link-layer frame appropriate to the next link.
5. At subsequent routers, the link-layer addresses can change again. Under ordinary forwarding without NAT, the source and destination IP addresses remain the same, although fields such as IPv4 TTL / IPv6 Hop Limit change.
6. The destination host eventually receives the packet and processes the TCP destination port `443`.

```text
ROUTED CONNECTION — FIRST HOP

Laptop                  Router / default gateway          Remote server
192.168.1.25           192.168.1.1                       203.0.113.10
     |                         |                                 |
     +--- Ethernet frame ----->|---- further routed hops ------>|

FIRST-HOP ETHERNET:
  Source MAC:       02:11:22:33:44:55  (laptop)
  Destination MAC:  02:66:77:88:99:AA  (router)

ENCAPSULATED IP:
  Source IP:        192.168.1.25
  Destination IP:   203.0.113.10      (remote server)

ENCAPSULATED TCP:
  Source port:      51514
  Destination port: 443
```

**Notice the critical difference:** the laptop does **not** send the first Ethernet frame to the remote server's MAC. It sends the frame to its next hop, the router. The remote server's IP remains the packet's intended destination.

**Real-world complication:** A home or office gateway often performs NAT, which can rewrite the source IP and possibly source port. That does not change the basic distinction among IP routing, per-link framing, and transport-layer ports.

## Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| IP address | Network-layer address used in packet delivery | `192.168.1.25` |
| IPv4 / IPv6 | Two versions of Internet Protocol addressing | 32-bit vs. 128-bit addresses |
| MAC address | Link-layer interface address on networks such as Ethernet | `02:11:22:33:44:55` |
| Port | Numeric identifier used by TCP or UDP endpoints | TCP destination port `443` |
| Socket endpoint | Transport protocol plus local IP address and port, in common networking usage | `TCP 192.168.1.25:51514` |
| ARP | IPv4 address-to-link-layer address resolution | Finding the gateway's MAC |
| Neighbor Discovery (ND) | IPv6 neighbor and router discovery mechanisms using ICMPv6 | Neighbor Solicitation / Advertisement |
| Subnet | IP network grouping determined by address and prefix | `192.168.1.0/24` |
| Default gateway | Router used for destinations without a more specific route | `192.168.1.1` |
| Ethernet frame | Link-layer unit carrying a payload across Ethernet | Contains source/destination MAC fields |
| IP packet | Network-layer unit carrying transport or other payload | Contains source/destination IP fields |
| TCP segment / UDP datagram | Transport-layer unit | Contains source/destination port fields |
| NAT | Translation of IP addresses, sometimes ports, at a network boundary | Private host connecting to the Internet |

## Real-world cybersecurity application: interpret a packet capture

You're examining traffic from a workstation that connected to an unfamiliar external server. A packet captured on the workstation's Ethernet interface contains these illustrative fields:

| Field | Observed value |
|---|---|
| Ethernet source | `02:11:22:33:44:55` |
| Ethernet destination | `02:66:77:88:99:AA` |
| IPv4 source | `192.168.1.25` |
| IPv4 destination | `203.0.113.10` |
| TCP source port | `51514` |
| TCP destination port | `443` |
| TCP flag | `SYN` |

**What can you reasonably infer?**

- The workstation is attempting to initiate a TCP connection to `203.0.113.10:443`.
- The Ethernet frame is directed to the local next-hop interface, which could be the default gateway. You would verify that mapping with the host's neighbor table or network configuration.
- The destination port is commonly associated with HTTPS, but **port 443 alone does not prove that HTTPS is being used**, nor that the activity is benign or malicious.
- A **SYN** alone does not prove the TCP connection was established. Look for the SYN-ACK and final ACK, or other evidence of the outcome.

**Defensible conclusion:** This capture shows a TCP connection attempt toward a remote IP address, using the local next hop for Ethernet delivery. More packets and context are needed to determine the application, whether the connection succeeded, and whether it is suspicious.

### Optional hands-on practice (authorized traffic only)

In Wireshark, capture traffic on a network you own or are explicitly authorized to inspect. Select a TCP packet and expand:

1. **Ethernet II** — source and destination MAC addresses.
2. **Internet Protocol Version 4/6** — source and destination IP addresses.
3. **Transmission Control Protocol** — source and destination ports and flags.

Try the display filter:

```wireshark
 tcp.flags.syn == 1
```

Then compare packets from a local-network conversation with packets destined for a routed network. Ask: *Does the Ethernet destination match the final IP destination's interface, or the local router's interface?* Actual captures depend on the interface, link type, and capture location.

## Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “The destination MAC always belongs to the final server.” | On a routed path, the destination MAC generally belongs to the **next hop on the current link**. |
| “A MAC address uniquely and permanently identifies a device.” | MACs can be randomized, locally administered, changed, or spoofed; a device can have multiple interfaces. |
| “IP addresses are always permanent and globally unique.” | Addresses can be private, temporary, dynamically assigned, or translated; uniqueness depends on network scope. |
| “Port 443 proves that traffic is HTTPS.” | Port numbers suggest common uses, but applications can use nonstandard ports and protocols. |
| “ARP finds the MAC address of any Internet server.” | ARP resolves IPv4 neighbors on a local link; routed traffic uses the **next hop's** link-layer address. |

## Check your understanding

1. **Identify:** Which address type is used for routing between IP networks, which is used for delivery on a MAC-addressed local link, and which helps identify a TCP/UDP endpoint?
2. **Explain:** A laptop is connecting to a server on the same IPv4 subnet. What information might it need from ARP before sending an Ethernet frame?
3. **Scenario:** A laptop sends a TCP SYN to a remote web server. The Ethernet destination MAC belongs to the router, but the destination IP belongs to the web server. Is this normal? Explain why.
4. **Compare:** How does IPv6 Neighbor Discovery differ from describing it simply as “ARP for IPv6”?
5. **Interpret:** A capture shows destination TCP port `443`. What does that suggest, and what can it **not** prove?
6. **Scenario:** Two connections from the same laptop reach the same server on TCP port `443`, but use different source ports. Why can the server distinguish the connections?
7. **Reason it through:** A packet leaves a laptop, crosses two routers, and reaches a server. Which addressing information typically changes at each routed Ethernet hop, and which destination information normally stays the same? Name an important exception.

## Answer key with explanations

<details>
<summary>Show answers and explanations</summary>

1. **IP addresses** support network-layer routing; **MAC addresses** direct frames on relevant local links; **TCP/UDP ports** identify transport-layer endpoints. Each operates at a different layer.
2. It may need the **server's MAC address**, resolved from the server's IPv4 address, unless it already has a usable cache entry. That lets it address the local Ethernet frame.
3. **Yes.** For a remote destination, the host sends the Ethernet frame to its **next-hop router**, while keeping the remote server as the IP destination.
4. IPv6 ND performs address resolution **and additional functions**, including router discovery and reachability-related functions. It uses ICMPv6 messages rather than IPv4 ARP messages.
5. It **suggests** a service commonly associated with HTTPS. It does **not prove** HTTPS, successful connection establishment, legitimate traffic, or the identity of the application.
6. A TCP connection is commonly identified by the combination of **source IP, source port, destination IP, and destination port** (with TCP as the transport protocol). Different source ports distinguish these connections.
7. On Ethernet links, **source and destination MAC addresses normally change at each routed hop** as new frames are created. The **destination IP normally stays the remote server's IP**. An important exception is **NAT or another address-translating intermediary**, which can rewrite IP addresses and sometimes ports.

</details>

## Teach it back in 60 seconds

Without looking at the diagram, explain aloud or in writing:

1. What different jobs do IP addresses, MAC addresses, and ports perform?
2. Why does a laptop send a frame to its router's MAC when connecting to a remote IP?
3. What does ARP do, and what is the corresponding IPv6 mechanism?
4. Why can't you conclude that traffic is safe or even HTTPS just because its destination port is `443`?

If you can trace one packet and explain the different addresses without mixing up their scopes, you have the core mental model.

## Key takeaways

- **IP = network-layer destination; MAC = local link delivery; port = transport-layer endpoint.**
- **Local destination:** a host can resolve the destination's local link-layer address and send directly.
- **Routed destination:** a host addresses the frame to the **next hop**, while the IP packet targets the remote host.
- **ARP handles IPv4 local address resolution; IPv6 Neighbor Discovery performs address resolution and more.**
- **Interpret addresses and ports together.** One field rarely establishes the application, intent, or outcome of a connection by itself.

## Further learning and references

Primary technical references:

- [RFC 791 — Internet Protocol (IPv4)](https://www.rfc-editor.org/rfc/rfc791)
- [RFC 8200 — Internet Protocol, Version 6 (IPv6) Specification](https://www.rfc-editor.org/rfc/rfc8200)
- [RFC 826 — An Ethernet Address Resolution Protocol (ARP)](https://www.rfc-editor.org/rfc/rfc826)
- [RFC 4861 — Neighbor Discovery for IP version 6 (IPv6)](https://www.rfc-editor.org/rfc/rfc4861)
- [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293)
- [RFC 768 — User Datagram Protocol (UDP)](https://www.rfc-editor.org/rfc/rfc768)

**Related guide:** [04 — The TCP Three-Way Handshake](04-tcp-three-way-handshake.md), for a closer look at SYN, SYN-ACK, and ACK.

---

© 2026 GhostParticle404.
