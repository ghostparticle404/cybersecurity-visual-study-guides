![DHCP DORA process visual study guide showing Discover, Offer, Request, Acknowledge, lease settings, and a hotel front desk analogy](../00-visual-guides/01.07.DHCP%20DORA%20Process%20Visual%20Study%20Guide.png)

# 07 — DHCP: How Devices Get IP Addresses

**Cybersecurity Visual Study Guides · 01 — Networking Fundamentals**  
**Estimated learning time:** 20–30 minutes, including practice

You join a Wi-Fi network, and moments later your laptop can communicate with other devices. Who decided which IP address it should use? In many IPv4 networks, **Dynamic Host Configuration Protocol (DHCP)** handles that negotiation. This guide explains the familiar **DORA** exchange, what a lease actually means, and how to investigate the settings your device received.

## Why this matters

A device can connect to a Wi-Fi access point yet still fail to reach the internet. The problem might be a missing address, incorrect subnet mask, unusable default gateway, or bad DNS server information. DHCP is one of the first places a network troubleshooter looks. From a security perspective, unexpected DHCP responses can also reveal misconfiguration—or a potentially unauthorized DHCP server.

## Learning objectives

By the end, you should be able to:

1. **Sequence** the four messages in a typical initial DHCPv4 lease: Discover, Offer, Request, and Acknowledge.
2. **Explain** what an address lease is and identify common configuration options.
3. **Distinguish** DHCP client, server, and relay roles.
4. **Inspect** DHCP settings on your own device and interpret their implications.
5. **Trace** a basic DHCP exchange and recognize evidence that warrants investigation.

## The big picture

**DHCP** automates the delivery of IP configuration. On a typical IPv4 network, a client requests configuration and a DHCP server offers an address from a managed pool, along with parameters such as a subnet mask, default gateway, and DNS servers. The address is commonly **leased** for a limited period rather than assigned permanently.

DHCP does **not** itself prove that a device or server is trustworthy, guarantee internet access, or replace routing and DNS.

## The memorable hook: a hotel front desk

Imagine arriving at a hotel where you don't have a room number.

You call out in the lobby: **“Is there a front desk that can check me in?”** (*Discover*.)

A receptionist replies: **“Room 214 is available, and here's the information you'll need.”** (*Offer*.)

You answer: **“I'd like room 214, please.”** (*Request*.)

The receptionist confirms: **“It's yours until checkout tomorrow.”** (*Acknowledge*.)

You can now find your room, navigate the building, and use its services. But your room assignment has an expiration date. Before it expires, you may be able to extend your stay—much as a DHCP client can renew a lease.

### Where the analogy stops working

A real DHCP client exchanges network messages, not spoken requests. Several DHCP servers may offer addresses; a client generally selects one and identifies it in its request. Acknowledgment is not authentication. DHCP also distributes configuration beyond the address itself, and the initial four-message pattern is **not** the only DHCP exchange: renewal can use a shorter sequence.

| Hotel story | Technical concept | Why it matters |
|---|---|---|
| Guest | DHCP client | Needs configuration to communicate |
| Reception desk | DHCP server | Offers and manages address leases |
| Available room | Offered IPv4 address | Must fit the network's allocation rules |
| Hotel directions | Gateway, subnet mask, DNS options | Help the client communicate beyond itself |
| Checkout time | Lease expiration | The assignment is time-limited |
| Extending the stay | Lease renewal | May retain the existing address |

## How DHCPv4 actually works: DORA

The classic initial lease exchange is **DORA**:

```text
DHCP client                               DHCP server
     |                                         |
     | ---- DHCPDISCOVER --------------------> |  1. Who can configure me?
     | <---------------------- DHCPOFFER ----- |  2. Here's an offered address.
     | ---- DHCPREQUEST ---------------------> |  3. I select this offer.
     | <----------------------- DHCPACK ------ |  4. Lease and options confirmed.
     |                                         |
```

**1. Discover.** A client without an IPv4 address typically broadcasts a `DHCPDISCOVER` message to find DHCP servers. It uses UDP, normally **source port 68** to **destination port 67**.

**2. Offer.** One or more servers may respond with `DHCPOFFER`, proposing an IPv4 address and configuration options. An offer is **not yet a completed lease**.

**3. Request.** The client sends `DHCPREQUEST` to indicate which offer it selected. During initial selection this is commonly broadcast, helping other offering servers recognize that their offers were not chosen.

**4. Acknowledge.** The selected server sends `DHCPACK` to confirm the lease and parameters. The client can then complete its configuration, subject to checks such as address-conflict detection. A server can instead send `DHCPNAK` in certain cases, and the client must restart configuration.

### What a lease can provide

| Configuration | Example (illustrative) | Purpose |
|---|---|---|
| IPv4 address | `192.168.1.100` | Client's assigned address |
| Subnet mask | `255.255.255.0` | Determines the local IPv4 subnet |
| Default gateway | `192.168.1.1` | Next hop for destinations outside the local subnet |
| DNS resolver | `192.168.1.1` | Server to ask about domain names |
| Lease duration | 8 hours | Time the address allocation remains valid |
| DHCP server identifier | `192.168.1.1` | Identifies the server associated with the lease |

These are examples, not universal defaults. The router, DNS resolver, and DHCP server may be different devices. Some networks intentionally omit particular options.

### Lease renewal and expiration

A DHCP lease is not a promise that the client will keep the same address forever. In common DHCPv4 configurations, the client tries to **renew** at **T1**, typically 50% of the lease duration, and **rebind** at **T2**, typically 87.5%. Those times can be configured differently. During renewal, the client commonly sends a DHCPREQUEST directly to its leasing server; during rebinding it broadcasts to reach any suitable server. A successful DHCPACK extends the lease. If a lease expires without renewal, the client must stop using that leased address.

### What if the server is on another subnet?

Initial DHCPv4 discovery uses broadcasts, and routers generally do not forward ordinary broadcasts between subnets. A **DHCP relay agent** can receive the client's request and forward it to a server elsewhere, then relay the response back. This is why a large organization can use centralized DHCP infrastructure without placing a server in every VLAN.

### IPv6 is different

**DORA is the familiar DHCPv4 initial allocation sequence, not the DHCPv6 message sequence.** IPv6 hosts may use Stateless Address Autoconfiguration (SLAAC), DHCPv6, or both depending on network configuration. DHCPv6 uses different messages and ports (UDP 546/547). Do not assume every IPv6 address came from DHCP.

## Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| DHCP | Dynamic Host Configuration Protocol | Automates IP configuration |
| DHCP client | Device requesting configuration | Laptop joining a network |
| DHCP server | Service managing offers and leases | Router or enterprise server |
| Discover | Client searches for DHCP servers | `DHCPDISCOVER` |
| Offer | Server proposes configuration | `DHCPOFFER` |
| Request | Client requests a chosen configuration | `DHCPREQUEST` |
| Acknowledge | Server confirms the lease | `DHCPACK` |
| Lease | Time-limited assignment | May need renewal |
| Address pool / scope | Set of addresses a server can allocate | `192.168.1.100–192.168.1.200` |
| DHCP relay | Forwards DHCP traffic across subnet boundaries | Centralized server for multiple VLANs |
| Rogue DHCP server | Unauthorized server responding to clients | Can supply misleading gateway/DNS settings |
| APIPA / link-local IPv4 | Self-assigned `169.254.0.0/16` address in some environments | May indicate no usable DHCP lease, but is not proof of the cause |

## Real-world cybersecurity application: inspect your DHCP settings

**Situation:** A workstation connects to the office Wi-Fi, but websites do not load. A second device works normally.

**Evidence to collect:** Assigned IP address, subnet mask, default gateway, DNS resolver, DHCP server, lease times, and—if authorized—DHCP packets from a capture. Compare these against known-good network settings.

### Windows

Open Command Prompt and run:

```powershell
ipconfig /all
```

Look for the active adapter's **DHCP Enabled**, **IPv4 Address**, **Default Gateway**, **DHCP Server**, **DNS Servers**, **Lease Obtained**, and **Lease Expires** fields.

### macOS

First identify your active network service in **System Settings → Network**. For a typical Wi-Fi interface, you can inspect the DHCP packet information with:

```bash
ipconfig getpacket en0
```

`en0` is an example interface; it may not be your active interface. You can check interface names using `networksetup -listallhardwareports`. The Network settings panel also shows current TCP/IP and DNS information.

### Linux

Check current addresses and routes:

```bash
ip -4 addr
ip -4 route
```

These commands show configured addressing and routing, **not necessarily which DHCP server supplied them**. For DHCP lease/server details, inspect your network manager's connection information or lease records; exact commands and locations vary by distribution.

### If you have an authorized packet capture

In Wireshark, try this display filter:

```text
bootp
```

DHCPv4 is commonly decoded under the historical `bootp` filter name. Inspect the **DHCP Message Type** option, transaction ID, offered address (`yiaddr`), server identifier, lease time, router, and DNS options. Follow the same transaction ID across the exchange; do not assume every four adjacent DHCP packets belong to the same client.

**Reasoning:** If the device has an unexpected gateway or DNS server, investigate whether the DHCP server is approved and whether the configuration matches policy. An unfamiliar server can indicate a rogue server, but it can also reflect a legitimate guest network, a newly deployed service, or a configuration change.

**Defensible conclusion:** DHCP evidence helps narrow the cause; it does not, by itself, establish malicious intent. Validate against trusted network documentation and additional logs.

> **Safety:** Inspect only your own device or networks you are authorized to administer. Do not deploy a DHCP server or send disruptive DHCP traffic on a shared network as a learning exercise.

## Common misconceptions

| Myth | More accurate explanation |
|---|---|
| “DHCP gives every device a permanent IP.” | Leases are time-limited; addresses may change. |
| “An offer means the client has its address.” | The server normally confirms an initial lease with DHCPACK. |
| “The DHCP server must be the router.” | It can be a separate host, reachable through a relay. |
| “DORA happens every time a lease renews.” | Renewals often use a shorter DHCPREQUEST/DHCPACK exchange. |
| “If DHCP succeeds, the internet must work.” | Routing, DNS, firewalls, upstream connectivity, and other factors can still fail. |

## Check your understanding

1. Expand **DORA** and put its four messages in order.
2. Why does a DHCP client need a lease rather than simply choosing any IPv4 address?
3. What is the difference between a `DHCPOFFER` and a `DHCPACK`?
4. **Scenario:** A client has `192.168.20.40/24`, but its default gateway is set to `192.168.10.1`. What would you investigate, and why?
5. **Scenario:** A packet capture contains offers from two different DHCP servers. Does that prove an attack? Explain your reasoning.
6. How can a client on VLAN 20 obtain a lease from a DHCP server on another subnet?
7. A client has an eight-hour lease and begins renewing after about four hours. Is that automatically an error? Explain the purpose of renewal.

<details>
<summary>Show answers and explanations</summary>

1. **Discover → Offer → Request → Acknowledge.** The client finds servers, receives a proposed configuration, requests a selected offer, and receives confirmation.
2. **To coordinate address allocation.** The server manages a pool and lease state, reducing conflicts and distributing consistent settings. DHCP alone cannot eliminate every conflict.
3. **An offer proposes; an acknowledgment confirms.** The client should not treat an initial offer alone as an established lease.
4. **Check the DHCP router option and subnet configuration.** With a `/24` mask, `192.168.10.1` is outside the client's `192.168.20.0/24` subnet; an ordinary host generally cannot use that off-link gateway without additional special configuration. Compare against an approved configuration and inspect the lease.
5. **No.** Multiple servers can legitimately respond, though an unauthorized server is a security concern. Check server identifiers, network design, and expected DHCP infrastructure.
6. **Through a DHCP relay agent.** It forwards DHCP requests and responses between the client subnet and the remote server.
7. **No.** A common default is renewal at 50% of the lease lifetime (T1), so four hours is expected for an eight-hour lease unless configured otherwise. Renewal helps preserve the assignment before expiration.

</details>

## Teach it back in 60 seconds

Without looking above, explain:

- What happens in each DORA step, and which side initiates it?
- Why an address lease has a duration and what renewal accomplishes.
- How you would tell a DHCP configuration problem from a possible DNS or routing problem.
- Why DHCP relay is useful on networks with multiple VLANs.

## Key takeaways

1. **DORA describes a typical initial DHCPv4 lease:** Discover → Offer → Request → Acknowledge.
2. **DHCP supplies configuration, not just an IP address.** Subnet, gateway, DNS, and lease options can be equally important.
3. **A lease is temporary.** Clients may renew before expiration, often without repeating all four initial steps.
4. **DHCP relays extend reach across subnets.** The DHCP server does not need to sit on the client's local link.
5. **DHCP settings are useful evidence.** Unexpected servers or options deserve verification, not automatic accusations of compromise.

## Further learning and references

**Primary references:**

- [RFC 2131 — Dynamic Host Configuration Protocol](https://www.rfc-editor.org/rfc/rfc2131)
- [RFC 2132 — DHCP Options and BOOTP Vendor Extensions](https://www.rfc-editor.org/rfc/rfc2132)
- [RFC 8415 — Dynamic Host Configuration Protocol for IPv6 (DHCPv6)](https://www.rfc-editor.org/rfc/rfc8415)

**Optional practice:** On an authorized device, record your DHCP server, gateway, DNS settings, and lease duration. Explain how each setting helps the device communicate. If you have access to an approved DHCPv4 packet capture, identify the four DORA message types and compare their transaction IDs.

**Related guide:** [06 — DNS: What Happens When You Type a URL?](../01-networking-fundamentals/06-dns-url-resolution.md)

---

© 2026 GhostParticle404.
