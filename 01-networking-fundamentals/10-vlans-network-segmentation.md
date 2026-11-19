![Guide 10: VLANs and secure network segmentation, illustrating conference wristbands, access and trunk ports, and a three-VLAN office network](../00-visual-guides/01.10.VLANs_%20Secure%20Network%20Segmentation%20Guide.png)

# 10 — VLANs: Segmenting a Network

**Cybersecurity Visual Study Guides · 01 — Networking Fundamentals**  
**Estimated learning time:** 25–30 minutes, including practice

A company can put employees, visitors, and servers on the same physical switching infrastructure without putting them all in the same logical network. Virtual LANs (VLANs) make that possible. This lesson explains what the separation actually means, how switches handle VLAN traffic, and what must happen before devices in different VLANs can communicate.

## 1. Why this matters

Imagine a small office where staff laptops, visitor Wi-Fi, and internal servers all connect through the same switching infrastructure. If everything shares one flat network, it is harder to separate traffic and apply access policies. VLANs provide a way to organize devices into distinct Layer 2 networks without buying a separate physical switch for every group.

Security teams encounter VLANs when designing networks, investigating connectivity failures, reviewing firewall rules, and containing access to sensitive systems. **A VLAN creates a boundary for Layer 2 traffic; it does not, by itself, provide a complete security policy.**

## 2. Learning objectives

By the end of this lesson, you should be able to:

1. **Explain** how a VLAN separates Layer 2 broadcast domains on shared switching infrastructure.
2. **Distinguish** access ports from trunk ports and describe the role of IEEE 802.1Q tags.
3. **Trace** traffic between devices in the same VLAN and between devices in different VLANs.
4. **Design** a staff, guest, and server VLAN plan with suitable IP subnets and port assignments.
5. **Identify** where routing and access-control policies are needed to limit cross-VLAN traffic.

## 3. The big picture, in plain English

A **virtual local area network (VLAN)** is a logical Layer 2 network. A managed switch can assign ports or traffic to different VLANs so that devices in one VLAN are separated from the broadcast domain of another VLAN, even when they use the same physical equipment.

For example:

| Group | VLAN ID | Example IPv4 subnet | Intended access |
|---|---:|---|---|
| Staff | 10 | `192.168.10.0/24` | Internet and approved internal services |
| Guests | 20 | `192.168.20.0/24` | Internet only |
| Servers | 30 | `192.168.30.0/24` | Only approved administrative and application traffic |

**Important:** VLAN IDs and IP subnets are different concepts. Assigning VLAN 10 does not automatically create subnet `192.168.10.0/24`; an administrator configures the Layer 2 membership and Layer 3 addressing separately.

## 4. The memorable hook: conference wristbands

Picture a conference in one enormous venue. Everyone enters the same building, but their wristbands indicate which areas they may enter:

- **Blue wristband — Staff:** offices and staff work areas.
- **Yellow wristband — Guest:** public talks and the café.
- **Purple wristband — VIP:** a designated restricted area.

The building represents shared physical network infrastructure. The colored groups represent VLAN membership. People can be in the same building without being in the same access group.

Now imagine someone needs to move from the guest area into a staff-only area. They need to pass a checkpoint with an explicit access decision. That checkpoint is a useful mental model for **routing combined with a firewall or access-control policy**.

### Where the analogy stops working

A wristband is a human authorization credential; a VLAN tag is **not** an identity credential or a cryptographic security control. Switches use VLAN membership and forwarding logic to separate Layer 2 traffic. They do not inspect a person's permissions merely because a frame has a VLAN ID. Furthermore, a router can forward traffic between VLANs unless routing policies or security controls restrict it.

## 5. Translate the story into technical reality

| Story element | Technical concept | Why it matters |
|---|---|---|
| Shared conference venue | Shared physical switch infrastructure | Several logical networks can use the same hardware |
| Colored wristband group | VLAN membership / VLAN ID | Identifies a Layer 2 broadcast domain |
| Door assigned to one group | Access port | Usually carries traffic for one assigned VLAN |
| Corridor serving several groups | Trunk link | Can carry frames for multiple VLANs |
| Security checkpoint between areas | Inter-VLAN routing plus filtering policy | Allows selected traffic across VLAN boundaries |

The metaphor helps with **separation**. The real network depends on Ethernet frames, switch forwarding tables, VLAN configuration, IP addressing, routing, and policy enforcement.

## 6. How it actually works — step by step

### Step 1: Create the VLANs

On a managed switch, an administrator defines VLANs such as 10 (staff), 20 (guest), and 30 (servers). VLAN numbers are identifiers; the friendly names are labels chosen by the administrator.

Each VLAN has its own Layer 2 broadcast domain. A normal broadcast frame originating in VLAN 20 is forwarded only within VLAN 20, not into VLAN 10 or 30.

### Step 2: Assign access ports

A staff laptop may connect to a switch port configured as an **access port in VLAN 10**. Its ordinary Ethernet frames arrive untagged, and the switch associates them with VLAN 10 internally. Another port can be assigned to VLAN 20 for a guest device.

The device usually does not need to know its VLAN number. The switch's port configuration determines its VLAN membership.

### Step 3: Carry multiple VLANs across a trunk

Suppose two managed switches need to carry staff and guest traffic between them. A **trunk** can transport frames from multiple VLANs over one physical link.

IEEE **802.1Q** defines VLAN tagging in Ethernet frames. For tagged traffic, the tag carries a VLAN identifier, allowing the receiving switch to associate each frame with the correct VLAN. On common 802.1Q trunks, a configured **native VLAN** may carry untagged frames; native-VLAN behavior and configuration must be checked on both ends.

```text
Switch A                     Switch B
VLAN 10 (staff) --\       /-- VLAN 10 (staff)
                  [802.1Q trunk]
VLAN 20 (guest) --/       \-- VLAN 20 (guest)

Tagged frames on the trunk identify their VLAN.
```

**Technical detail:** The 802.1Q tag includes a 12-bit VLAN ID field. IDs 1–4094 are usable VLAN identifiers in the standard tagging scheme, with some IDs reserved or handled specially by vendors. VLAN 0 indicates priority-tagged traffic rather than membership in VLAN 0; VLAN 4095 is reserved.

### Step 4: Forward within a VLAN

Two staff devices in VLAN 10 and the same IP subnet can typically communicate through Layer 2 switching. The switch learns source MAC addresses and associates them with VLANs and ports, then forwards known unicast traffic toward the appropriate port within that VLAN.

If the destination MAC is unknown, the switch may flood the frame to eligible ports **within that VLAN**. Broadcast traffic is also contained within the VLAN, subject to additional switch features and policies.

### Step 5: Route between VLANs, when needed

A staff laptop in `192.168.10.0/24` cannot reach a server in `192.168.30.0/24` using ordinary same-subnet Layer 2 forwarding. It normally sends the packet to a **default gateway**. A router or Layer 3 switch can then route between the subnets.

A common design gives each VLAN a Layer 3 interface, such as a switched virtual interface (SVI):

| VLAN | Example gateway |
|---|---|
| Staff 10 | `192.168.10.1` |
| Guest 20 | `192.168.20.1` |
| Servers 30 | `192.168.30.1` |

The routing device needs appropriate interfaces, routing configuration, and policies. **Inter-VLAN routing can provide connectivity; it does not automatically enforce least privilege.**

### Step 6: Apply explicit access rules

The security design might allow staff to reach a designated application server on TCP port 443, deny guest access to internal subnets, and allow guest traffic to approved internet destinations. Depending on the equipment, these rules may be enforced by a firewall, router ACLs, Layer 3 switch ACLs, or other controls.

**Example traffic traces**

```text
Staff laptop (VLAN 10) -> Staff printer (VLAN 10)
  Same VLAN/subnet -> switched locally (subject to policy)

Staff laptop (VLAN 10) -> Application server (VLAN 30)
  Default gateway -> inter-VLAN routing -> policy check -> server

Guest laptop (VLAN 20) -> Application server (VLAN 30)
  Default gateway -> policy check -> DENY (per office policy)
```

These traces describe a configured design, not automatic behavior guaranteed by creating VLANs.

## 7. Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| **VLAN** | Virtual Local Area Network; a logical Layer 2 broadcast domain | VLAN 20 for guests |
| **VLAN ID (VID)** | Numeric identifier used to distinguish VLANs | 10, 20, 30 |
| **IEEE 802.1Q** | Ethernet VLAN-tagging standard | Tagged frames on a trunk |
| **Access port** | Switch port ordinarily assigned to one VLAN for endpoint traffic | Staff desktop port in VLAN 10 |
| **Trunk port** | Link configured to carry multiple VLANs | Switch-to-switch uplink |
| **Native VLAN** | VLAN associated with untagged traffic on certain 802.1Q trunk configurations | Must be configured consistently |
| **Broadcast domain** | Set of interfaces that can receive a Layer 2 broadcast | One VLAN is normally one broadcast domain |
| **Inter-VLAN routing** | Layer 3 forwarding between VLAN-associated IP networks | Staff-to-server connection |
| **SVI** | Switched Virtual Interface; a Layer 3 interface associated with a VLAN | `192.168.20.1` as a gateway |
| **ACL** | Access Control List; permit/deny rules applied to traffic | Block guest-to-staff access |
| **Segmentation** | Dividing networks into controlled zones | Separate guest and staff devices |

## 8. A realistic cybersecurity application: small-office segmentation

### Situation

An office has employee laptops, visitor Wi-Fi, and a file/application server. The administrator wants visitors to use the internet but never access internal employee or server subnets. Staff should reach one application on the server, but not have unrestricted access to every server service.

### Proposed topology

```text
                     Internet
                        |
                 [Edge firewall]
                        |
              [Router / L3 switch]
               /        |        \
          VLAN 10    VLAN 20    VLAN 30
           Staff      Guest      Servers
       192.168.10/24  .20/24     .30/24
               \        |        /
              [Managed switching]
              (access ports / trunks)
```

*Diagram note:* Each VLAN's subnet is `192.168.X.0/24`; the abbreviated labels above save space. The physical and logical paths vary by implementation. In many deployments, a single firewall performs both routing and filtering.

### Design decisions

| Source | Destination | Example policy | Reason |
|---|---|---|---|
| Staff VLAN 10 | Approved server in VLAN 30, TCP 443 | Allow | Required business application |
| Staff VLAN 10 | Other server services | Deny unless needed | Limit unnecessary access |
| Guest VLAN 20 | Staff VLAN 10 | Deny | Prevent visitor access to staff devices |
| Guest VLAN 20 | Server VLAN 30 | Deny | Protect internal services |
| Guest VLAN 20 | Internet | Allow per egress policy | Provide visitor connectivity |

The policy must be applied **at a point where the traffic actually passes**. An ACL on the wrong interface or a firewall bypassed by local inter-VLAN routing will not achieve the intended separation. Internet access also depends on appropriate upstream routing, DNS, and often network address translation (NAT), depending on the addressing design.

### Evidence and investigation

A guest reports that an internal server is unreachable. An analyst checks:

1. The guest device's IP address and default gateway.
2. The switch port or Wi-Fi SSID's VLAN assignment.
3. Whether the relevant trunk carries VLAN 20.
4. Whether the guest can reach its gateway.
5. Whether a firewall/ACL rule denies guest-to-server traffic.

**Defensible conclusion:** If logs show a policy deny matching the guest's source IP and server destination, the failed connection may be **expected and correct**. A timeout alone would not prove that a firewall blocked it; missing routes, incorrect tagging, host configuration, and service availability can produce similar symptoms.

## 9. Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “A VLAN is a separate physical network.” | VLANs are logical Layer 2 networks that can share physical switching infrastructure. |
| “VLAN 10 automatically uses `192.168.10.0/24`.” | VLAN IDs and IP subnets are independently configured. |
| “VLANs automatically stop all cross-VLAN access.” | Inter-VLAN routing can connect them; policy controls are needed to restrict traffic. |
| “Every trunk frame must be tagged.” | Native-VLAN conventions may allow untagged traffic on an 802.1Q trunk. |
| “A VLAN tag proves a device is trusted.” | Tags identify VLAN context; they are not authentication or encryption. |

**Additional design caution:** Misconfigured trunks, inappropriate native VLAN settings, overly permissive routing, and unnecessary trunk-allowed VLANs can undermine a segmentation plan. Some switch features and vendor implementations differ; validate behavior on the actual equipment.

## 10. Check your understanding

Try these without looking back at the glossary.

1. **Recall:** What is the main Layer 2 separation created by a VLAN?
2. **Compare:** How does an access port differ from a trunk port?
3. **Explain:** What information does an 802.1Q VLAN tag provide, and what does it *not* prove?
4. **Scenario:** A staff laptop and a printer are both in VLAN 10 and `192.168.10.0/24`. Under ordinary conditions, must their traffic pass through an inter-VLAN router? Explain.
5. **Scenario:** A guest laptop in VLAN 20 can reach its gateway but not a server in VLAN 30. Name **three** possible explanations and one piece of evidence you would seek for each.
6. **Design:** Assign VLAN IDs, IPv4 `/24` subnets, and typical access/trunk port roles for staff devices, guest Wi-Fi, and servers in a small office. Which traffic would you allow between groups?
7. **Reasoning:** A network engineer enables routing between VLAN 10 and VLAN 30. A manager says, “Great—now VLAN 30 is secure because it's separate.” What important distinction would you explain?

## 11. Answer key with explanations

<details>
<summary>Show answers and explanations</summary>

**1.** A VLAN creates a distinct **Layer 2 broadcast domain**. Ordinary Layer 2 broadcasts in one VLAN are not forwarded into another VLAN.

**2.** An **access port** ordinarily carries endpoint traffic associated with one VLAN, typically untagged on the endpoint-facing link. A **trunk port** can carry multiple VLANs, generally using 802.1Q tags to identify them. Vendor-specific modes and configurations may vary.

**3.** The tag identifies the VLAN context of a tagged Ethernet frame (and includes priority-related fields). It does **not** authenticate the sender, encrypt traffic, or prove authorization.

**4.** No. If both devices are in the same VLAN and IP subnet, their traffic is ordinarily switched at Layer 2 without inter-VLAN routing. Other controls, such as isolation features or host firewalls, may still affect connectivity.

**5.** Examples: (a) a firewall/ACL intentionally denies VLAN 20 → VLAN 30; inspect matching policy logs, (b) routing toward VLAN 30 is missing or wrong; inspect route tables and interface status, (c) the server is offline or its host firewall blocks traffic; check server health and host logs. A VLAN or trunk misconfiguration is another possibility, although the guest's successful gateway access narrows the possibilities for its own local path.

**6.** One valid design is staff VLAN 10 (`192.168.10.0/24`), guest VLAN 20 (`192.168.20.0/24`), servers VLAN 30 (`192.168.30.0/24`). Endpoints use appropriate access VLANs; switch uplinks carrying several VLANs use trunks. Allow guests to approved internet destinations but deny internal subnets. Allow staff only the server services they require. The specific IDs and ranges are examples, not mandatory conventions.

**7.** Layer 2 separation is **not equivalent to a complete security boundary**. Once routing exists, traffic can cross VLANs unless firewall or ACL policy restricts it. The team should specify allowed flows, apply controls on the real traffic path, and test the resulting policy.

</details>

## 12. Teach it back in 60 seconds

Explain the design to someone who has never configured a switch. Without reading your notes, answer:

1. Why can three VLANs share one switch without becoming one broadcast domain?
2. What does an access port do differently from a trunk?
3. Why is a router or Layer 3 switch needed for typical communication between different VLAN subnets?
4. Why does the guest VLAN still need explicit access restrictions?

If you can explain all four using the conference wristbands **and** the actual networking terms, you have the core mental model.

## 13. Key takeaways

- **VLANs divide shared switching infrastructure into separate Layer 2 broadcast domains.**
- **Access ports** ordinarily serve one VLAN; **trunks** can carry several, often with **IEEE 802.1Q** tags.
- **VLAN IDs and IP subnet numbers are different configuration choices.**
- **Inter-VLAN routing enables communication between VLAN-associated IP networks; it does not automatically limit that communication.**
- **Effective segmentation requires both a sound VLAN design and correctly placed, tested access controls.**

## 14. Further learning and references

For publication and deeper study, consult primary documentation relevant to the hardware and software you use:

- **IEEE Std 802.1Q** — the authoritative standard for bridges and bridged networks, including VLAN tagging. Check the applicable edition through the IEEE 802.1 standards catalog.
- **Switch vendor configuration documentation** — VLAN creation, access/trunk modes, allowed VLAN lists, and native VLAN behavior for your specific model and software version.
- **Firewall / Layer 3 switch documentation** — inter-VLAN routing, SVIs, ACL evaluation, and logging behavior.

**Optional safe practice:** In an authorized lab or network simulator, configure two VLANs on a managed switch, test same-VLAN connectivity, add inter-VLAN routing, and then apply a rule blocking guest-to-staff traffic. Record what changes at each step. Never test VLAN misconfigurations or bypass techniques on networks without permission.

**Related guides:**

- [08 — Subnetting & CIDR: Visualized](08-subnetting-and-cidr.md)
- [09 — Switches, Routers, Firewalls & Gateways](09-switches-routers-firewalls-gateways.md)

---

© 2026 GhostParticle404.
