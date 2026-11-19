![Subnetting and CIDR infographic showing IPv4 subnet boundaries, /24 and /26 worked examples, and an apartment-building analogy](../00-visual-guides/01.08.Subnetting%26CIDR.png)

# 08 — Subnetting & CIDR: Visualized

**Series:** Cybersecurity Visual Study Guides  
**Category:** 01 — Networking Fundamentals  
**Estimated time:** 25–30 minutes, including practice

A network address is more than a collection of numbers separated by dots. Its prefix tells you which addresses belong together, where one subnet ends, and where the next begins. This lesson turns those boundaries into something you can calculate, visualize, and use during security investigations.

## Why this matters

Imagine reviewing firewall logs for a company that separates employee laptops, servers, and smart devices. You see a rule allowing traffic from `192.168.10.64/26`. Does that rule cover one machine, the whole office, or a specific group of addresses? Understanding subnetting lets you interpret the scope of the rule instead of guessing.

Subnet calculations also help you troubleshoot routing, interpret network scans, plan address allocations, and recognize when traffic crosses a network boundary.

## Learning objectives

By the end, you should be able to:

1. **Interpret** IPv4 CIDR prefixes and distinguish network bits from host bits.
2. **Calculate** the total number of addresses and conventional usable-host counts in `/24` and `/26` subnets.
3. **Determine** network addresses, broadcast addresses, and usable host ranges.
4. **Identify** the four `/26` networks contained in a `/24` network.
5. **Apply** subnet boundaries to a realistic firewall or security-monitoring question.

## The big picture, in plain English

An **IPv4 address** contains 32 bits, usually written as four decimal numbers called *octets*: `192.168.10.70`. Each octet represents eight bits.

**Subnetting** divides an address block into smaller address blocks. **Classless Inter-Domain Routing (CIDR)** notation writes an address followed by a slash and a **prefix length**, such as `192.168.10.0/24`. The prefix length tells you how many of the 32 bits identify the network portion.

- `/24` means **24 network bits** and **8 host bits**.
- `/26` means **26 network bits** and **6 host bits**.

The remaining host bits determine the addresses inside that block. A longer prefix means a **smaller** block, because fewer bits remain for host addresses.

> **Mental model:** The slash number marks the boundary. More bits reserved for the network means fewer addresses available inside it.

## A memorable story: apartment buildings and their boundaries

Picture an apartment complex occupying one city block. The complex has a street address, and each apartment has a number. The owner divides the property into four separate buildings, each with its own entrance and apartment-number range.

A `/24` is like the original large property: **256 numbered positions** in its address block. Dividing it into four `/26` blocks is like establishing four smaller buildings, each with **64 positions**. Each building has a clearly defined boundary. An apartment number in Building B does not suddenly become part of Building C just because the buildings are next to each other.

In a conventional IPv4 `/26` subnet, the first address identifies the subnet and the last is the directed broadcast address. That leaves 62 addresses ordinarily available for assignment to interfaces.

### Where the analogy stops working

An IP subnet is **not literally a building**, and IP addresses are not apartment mailboxes. Subnet boundaries are defined by binary prefixes, not physical walls. Devices in different subnets can communicate when routing and security policy allow it. Also, creating separate IP subnets does **not**, by itself, guarantee isolation: VLAN configuration, routing, access-control rules, and other controls determine what traffic is permitted.

## Translate the story into technical reality

| Story element | Real technical concept | Why it matters |
|---|---|---|
| Entire apartment complex | Larger address block, such as `192.168.10.0/24` | Defines the original range |
| Four separate buildings | Four `/26` subnets | Splits the larger block into smaller, nonoverlapping ranges |
| Building boundary | Network prefix / subnet boundary | Determines which addresses belong to a subnet |
| Apartment number | Host portion of an IPv4 address | Distinguishes addresses within a subnet |
| Building's official address | Network address | Identifies the subnet itself |
| Building-wide announcement | Directed broadcast address | Addresses the subnet's broadcast destination in conventional IPv4 networks |

Now let's calculate those boundaries instead of relying on the metaphor.

## How it actually works — step by step

### Step 1: Read the slash number

Consider `192.168.10.0/24`:

- IPv4 has **32 bits**.
- The prefix `/24` reserves **24 bits** for the network.
- That leaves **32 − 24 = 8 host bits**.

For `192.168.10.0/26`, the prefix reserves 26 network bits, leaving **6 host bits**.

### Step 2: Calculate the block size

For a prefix length of `p`, the number of addresses in the IPv4 block is:

```text
Host bits = 32 − p
Total addresses = 2^(host bits)
```

For conventional IPv4 subnets that reserve a network address and a broadcast address, the typical usable-host count is:

```text
Usable host addresses = total addresses − 2
```

| Prefix | Network bits | Host bits | Total addresses | Conventional usable hosts | Subnet mask |
|---|---:|---:|---:|---:|---|
| `/24` | 24 | 8 | 256 | 254 | `255.255.255.0` |
| `/25` | 25 | 7 | 128 | 126 | `255.255.255.128` |
| `/26` | 26 | 6 | 64 | 62 | `255.255.255.192` |
| `/27` | 27 | 5 | 32 | 30 | `255.255.255.224` |

**Important exception:** The familiar “subtract two” rule is not universal. IPv4 `/31` point-to-point links can use both addresses under RFC 3021; `/32` identifies a single address. The calculations in this lesson focus on conventional `/24` and `/26` LAN-style subnets.

### Step 3: Understand the bits

An octet is eight bits. A `/24` boundary falls after three complete octets:

```text
192       .168       .10        .0
11000000  .10101000  .00001010  .00000000
|---------- 24 network bits ----------| host |
```

For a `/26`, **two additional bits** in the fourth octet belong to the network:

```text
192       .168       .10        .0
11000000  .10101000  .00001010  .00|000000
|------------- 26 network bits --------|6 host bits
```

Those two extra network bits have four possible values: `00`, `01`, `10`, and `11`. This is why one `/24` can be divided into **four `/26` subnets**.

### Step 4: Work through a `/24`

**Given:** `192.168.10.0/24`

1. Host bits: `32 − 24 = 8`.
2. Total addresses: `2^8 = 256`.
3. Network address: `192.168.10.0`.
4. Broadcast address: `192.168.10.255`.
5. Conventional usable range: `192.168.10.1` through `192.168.10.254` (**254 addresses**).

The fourth octet ranges from `0` to `255`, inclusive.

### Step 5: Work through a `/26`

**Given:** `192.168.10.0/26`

1. Host bits: `32 − 26 = 6`.
2. Total addresses: `2^6 = 64`.
3. Network address: `192.168.10.0`.
4. Broadcast address: `192.168.10.63`.
5. Conventional usable range: `192.168.10.1` through `192.168.10.62` (**62 addresses**).

**Why does it end at `.63`?** A block containing 64 addresses begins at `.0`, so its last address is `.0 + 63`. The next `/26` begins at `.64`.

### Step 6: Find all four `/26` subnets inside a `/24`

Each `/26` occupies 64 consecutive addresses. Starting at `.0`, count by 64: **0, 64, 128, 192**.

| `/26` network | Full address range | Conventional usable host range | Broadcast |
|---|---|---|---|
| `192.168.10.0/26` | `.0–.63` | `.1–.62` | `.63` |
| `192.168.10.64/26` | `.64–.127` | `.65–.126` | `.127` |
| `192.168.10.128/26` | `.128–.191` | `.129–.190` | `.191` |
| `192.168.10.192/26` | `.192–.255` | `.193–.254` | `.255` |

All abbreviated ranges above use the same first three octets: `192.168.10`.

```text
192.168.10.0/24  (256 addresses)
│
├── 192.168.10.0/26      .0   – .63
├── 192.168.10.64/26     .64  – .127
├── 192.168.10.128/26    .128 – .191
└── 192.168.10.192/26    .192 – .255
```

**A fast boundary check:** `256 − 192 = 64`, using the final octet of the `/26` mask (`255.255.255.192`). This shortcut works for this example because the subnetting boundary is in the fourth octet. The binary-prefix method is the more general explanation.

### Step 7: Determine the subnet for an individual address

**Question:** Which `/26` contains `192.168.10.70`?

The fourth octet is `70`. It falls between **64 and 127**, so the subnet is:

- **Network:** `192.168.10.64/26`
- **Usable host range:** `192.168.10.65–192.168.10.126`
- **Broadcast:** `192.168.10.127`

A quick binary check confirms it: `70` is `01000110`, whose first two bits are `01`. That is the second `/26` block within the `/24`.

## Essential terminology and language of the field

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| **IPv4** | 32-bit Internet Protocol address format | `192.168.10.70` |
| **CIDR** | Classless Inter-Domain Routing; prefix-based way to express address blocks | `192.168.10.0/26` |
| **Prefix length** | Number of leading bits assigned to the network portion | `/26` means 26 network bits |
| **Subnet mask** | 32-bit mask marking network bits with 1s and host bits with 0s | `/26` = `255.255.255.192` |
| **Network address** | Address with all host bits set to zero | `192.168.10.64` for the second `/26` |
| **Host address** | Address that can identify an interface within the subnet, subject to addressing rules | `192.168.10.70` |
| **Broadcast address** | Address with all host bits set to one in a conventional IPv4 subnet | `192.168.10.127` for the second `/26` |
| **Subnet boundary** | The start/end of a prefix-defined address block | `/26` starts at `.0`, `.64`, `.128`, `.192` here |
| **Network segmentation** | Dividing networks into groups, usually paired with routing and access controls | Separate employee, server, and IoT subnets |

## A realistic cybersecurity application: interpreting a firewall rule

**Situation:** An analyst is reviewing an access-control rule that permits a management workstation to reach `192.168.10.64/26` on a particular service port. The inventory shows a server at `192.168.10.70` and a camera at `192.168.10.140`.

**Evidence:**

- Rule destination: `192.168.10.64/26`
- Server address: `192.168.10.70`
- Camera address: `192.168.10.140`

**Reasoning:**

1. A `/26` contains 64 addresses.
2. The block beginning at `.64` ends at `.127`.
3. The server at `.70` is inside that destination range.
4. The camera at `.140` belongs to the next `/26`: `192.168.10.128/26`.

**Defensible conclusion:** The rule's **destination-address condition** matches the server but not the camera. That does **not** prove the server is reachable: protocol, port, rule order, routing, device state, and other controls still matter. Nor does subnetting alone prove that the camera is securely isolated.

**Security lesson:** Correct subnet math prevents both overly broad access rules and mistaken conclusions during log review.

## Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “A `/26` is larger than a `/24` because 26 is bigger than 24.” | A longer prefix leaves **fewer host bits**, so a `/26` contains fewer addresses. |
| “Every IPv4 subnet has total addresses minus two usable addresses.” | That rule fits conventional subnet ranges such as `/24` and `/26`, but `/31` and `/32` require special treatment. |
| “The network address is always the first IP ending in `.0`.” | The network address depends on the **prefix**; `192.168.10.64` is a valid `/26` network address. |
| “Two different subnets cannot communicate.” | They can communicate through routing when policy permits. |
| “Subnetting automatically secures devices.” | Segmentation provides useful boundaries, but effective isolation depends on routing, filtering, and correct configuration. |

## Check your understanding

Try these without looking back at the worked examples. Show your reasoning for questions 4–7.

1. In IPv4, how many network bits and host bits does `/24` indicate?
2. How many **total** addresses are in a `/26`? How many are conventionally usable for hosts?
3. What is the subnet mask for `/26`?
4. **Scenario:** A device has address `192.168.10.200/26`. Identify its network address, broadcast address, and conventional usable host range. Explain how you found the boundary.
5. **Scenario:** A firewall rule allows destination `192.168.10.64/26`. Would the destination addresses `192.168.10.100` and `192.168.10.130` each match the rule's address range? Explain.
6. You divide `192.168.10.0/24` into equal `/26` subnets. List the four network addresses and explain why there are exactly four.
7. **Reasoning challenge:** A colleague says, “Our cameras are on a different subnet from employee laptops, so employees cannot reach them.” What additional information would you need before accepting that claim?

## Answer key with explanations

<details>
<summary>Show answers and explanations</summary>

**1.** `/24` means **24 network bits** and **8 host bits**. IPv4 addresses contain 32 bits in total, so `32 − 24 = 8`.

**2.** `/26` leaves six host bits, so `2^6 = 64` total addresses. In a conventional subnet, the network and broadcast addresses are not assigned to ordinary hosts, leaving **62 usable addresses**.

**3.** `255.255.255.192`. The first 24 mask bits are ones (`255.255.255`), and the next two are ones in the fourth octet (`11000000` = `192`).

**4.** `192.168.10.200` falls within `.192–.255`, so the network is **`192.168.10.192/26`**, the broadcast is **`192.168.10.255`**, and the conventional usable range is **`192.168.10.193–192.168.10.254`**. `/26` boundaries occur in steps of 64: `.0`, `.64`, `.128`, and `.192`.

**5.** `.100` **matches** the destination-address range because it falls in `.64–.127`. `.130` **does not match** because it belongs to `.128–.191`. This answers only the address-matching part of the firewall rule; it does not establish whether a connection is permitted overall.

**6.** The networks are **`192.168.10.0/26`**, **`192.168.10.64/26`**, **`192.168.10.128/26`**, and **`192.168.10.192/26`**. Increasing the prefix from `/24` to `/26` borrows two bits for subnet identification, yielding `2^2 = 4` equally sized subnets.

**7.** Check the **routing configuration**, **firewall/access-control policies**, **VLAN and Layer 3 design**, and whether other paths or permitted services connect the groups. Different IP subnets do not automatically prevent communication.

</details>

## Teach it back in 60 seconds

Without looking at the tables, explain:

1. What the slash number in `/26` tells you about network and host bits.
2. Why a `/26` contains 64 addresses but conventionally supports 62 assignable host addresses.
3. How you would find the subnet containing `192.168.10.70`.
4. Why different subnets are not necessarily isolated from one another.

If you can explain the **boundary**, the **block size**, and the **security implication**, you have the core mental model.

## Key takeaways

- **CIDR prefix length defines the boundary** between network bits and host bits.
- **Longer IPv4 prefixes create smaller address blocks:** `/26` contains 64 addresses; `/24` contains 256.
- **Subnet ranges follow binary boundaries**, not arbitrary decimal groupings. In this `/24`, `/26` networks begin at `.0`, `.64`, `.128`, and `.192`.
- **Network and broadcast addresses matter** when calculating conventional IPv4 usable-host ranges, with special cases such as `/31` and `/32`.
- **Subnetting informs security decisions**, but actual communication restrictions require routing and access-control analysis.

## Further learning and references

**Primary references**

- [RFC 4632 — Classless Inter-domain Routing (CIDR): The Internet Address Assignment and Aggregation Plan](https://www.rfc-editor.org/rfc/rfc4632) — CIDR addressing and prefix concepts.
- [RFC 950 — Internet Standard Subnetting Procedure](https://www.rfc-editor.org/rfc/rfc950) — Historical foundation for IPv4 subnet masks and subnetting.
- [RFC 3021 — Using 31-Bit Prefixes on IPv4 Point-to-Point Links](https://www.rfc-editor.org/rfc/rfc3021) — Explains the `/31` exception.

**Optional practice:** Choose a different private IPv4 `/24` block, divide it into four `/26` blocks, and calculate each network, broadcast, and conventional usable range. Verify that the four blocks are nonoverlapping and together cover all 256 addresses.

---

© 2026 GhostParticle404.
