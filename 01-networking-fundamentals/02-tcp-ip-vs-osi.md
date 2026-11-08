![Comparison of the seven-layer OSI reference model and four-layer TCP/IP model](../00-visual-guides/TCP_IP-vs.-OSI_TwoMaps_Same Network.png)

# 02 — TCP/IP vs. OSI: How the Models Compare

**Cybersecurity Visual Study Guides · 01 — Networking Fundamentals**  
**Estimated time:** 20–30 minutes, including practice  
**Prerequisite:** [01 — The OSI Model: Seven Layers Explained](01-osi-model.md)

> **Before publishing:** Replace the image filename in the first line with the exact name of your Guide 02 infographic. If it contains spaces, replace them with `%20` in the Markdown link. The image belongs in the repository's `00-visual-guides/` folder.

## Why this matters

A packet capture says **TCP**. A colleague says, “That's Layer 4.” A textbook calls TCP part of the **transport layer** in *both* models. Meanwhile, a diagram puts **HTTP** at OSI Layer 7 but in the TCP/IP **application layer**. Is somebody wrong?

Not necessarily. They're using two different ways to describe the same networking functions. Being able to translate between them helps you read documentation, interpret packet captures, troubleshoot connections, and communicate clearly with other security professionals.

## Learning objectives

By the end, you should be able to:

1. **Identify** all seven OSI layers and all four layers of the teaching version of the TCP/IP model.
2. **Map** each TCP/IP layer to the corresponding OSI layer or layers.
3. **Classify** common protocols and technologies using both models, while recognizing ambiguous cases.
4. **Explain** what *abstraction* means and why different models group functions differently.
5. **Apply** both models to reason about a failed website connection.

## The big picture: two maps of the same city

Imagine you're visiting a new city. You open two maps.

**Map A: The city planner's map.** It separates the city into seven kinds of detail: buildings, utilities, street access, traffic control, major roads, and so on. It's fantastic for asking, “Which part of this system is responsible for that?”

**Map B: The commuter's map.** It groups much of that detail into four practical views so you can figure out how to get from one place to another without first earning a degree in municipal engineering.

Both maps describe the **same city**. One isn't secretly hiding four neighborhoods. They simply group information differently.

That's the core distinction:

- **OSI** is a **seven-layer reference model** that separates communication responsibilities into finer conceptual categories.
- **TCP/IP** is a **protocol suite** commonly explained using a **four-layer model**: Application, Transport, Internet, and Link (also called Network Access).

**The point isn't to choose the “correct” map. It's to know which map helps answer your question.**

## 1. The world's least helpful city tour 🗺️

You ask a city planner how to reach a café.

The planner says: “First, we must discuss the distinction between public-facing services, information representation, conversational state, end-to-end delivery, route selection, local street access, and the actual pavement.”

You ask a commuter.

The commuter says: “Use the café app, take the express line, transfer at the junction, and walk the last block.”

Both are describing transportation. One gives you **more categories**; the other bundles several responsibilities into **broader functional groups**.

**Where the analogy stops working:** Networking layers aren't literal streets or city departments. The OSI and TCP/IP diagrams aren't two physical networks, and actual protocol implementations don't always fit neatly into either model. In particular, OSI's Session and Presentation functions usually aren't implemented as separate, universal layers in everyday TCP/IP stacks.

### Translate the story into technical reality

| Story element | Technical concept | Why it matters |
|---|---|---|
| Detailed city-planning map | Seven-layer OSI reference model | Helps isolate and discuss distinct responsibilities |
| Practical commuter map | Four-layer TCP/IP teaching model | Groups functions around widely deployed internet protocols |
| Same city, different map legends | **Abstraction** | A model deliberately simplifies reality for a purpose |
| Different groupings of streets | **Layer mapping** | Several OSI layers can correspond to one TCP/IP layer |
| Getting to the café | End-to-end communication | Different functions cooperate to deliver useful data |

## 2. The mapping: seven layers become four groups

This is the mapping to learn first:

| OSI layer | OSI function | TCP/IP layer | Typical examples |
|---|---|---|---|
| **7 — Application** | Network services used by applications | **Application** | HTTP, DNS, SMTP |
| **6 — Presentation** | Data representation, encoding, encryption concepts | **Application** | Serialization, character encoding, TLS-related functions* |
| **5 — Session** | Managing logical conversations | **Application** | Application session management* |
| **4 — Transport** | End-to-end transport between endpoints | **Transport** | TCP, UDP |
| **3 — Network** | Logical addressing and routing between networks | **Internet** | IPv4, IPv6, ICMP |
| **2 — Data Link** | Local framing and link-level delivery | **Link / Network Access** | Ethernet framing, Wi-Fi MAC functions |
| **1 — Physical** | Transmission of signals | **Link / Network Access** | Copper, fiber, radio transmission |

\* **Conceptual mapping, not a protocol's exclusive home.** For example, TLS is commonly described between application protocols and transport; labeling it “OSI Layer 6” is a teaching shortcut, not a universal implementation rule. Likewise, session-management features may be implemented by applications or protocols rather than a standalone OSI Layer 5 component.

### A compact text diagram

```text
         OSI (7 layers)                  TCP/IP (4 layers)
     ┌─────────────────────┐
  7  │ Application         │ ─┐
  6  │ Presentation        │ ─┼────────► Application
  5  │ Session             │ ─┘
     ├─────────────────────┤
  4  │ Transport           │ ──────────► Transport
     ├─────────────────────┤
  3  │ Network             │ ──────────► Internet
     ├─────────────────────┤
  2  │ Data Link           │ ─┐
  1  │ Physical            │ ─┴────────► Link / Network Access
     └─────────────────────┘
```

**Memory anchor:** **7 → 4**, because **3 top OSI layers are grouped into 1 TCP/IP Application layer**, and **2 bottom OSI layers are grouped into 1 TCP/IP Link layer**. The middle two map more directly.

## 3. How the two models describe one website request

Suppose you open `https://example.com` in your browser. The details depend on the connection and protocol version, but consider a common **HTTPS-over-TCP** example:

1. **Application:** The browser prepares an HTTP request. DNS may have been used to look up an IP address. HTTPS uses TLS to protect the HTTP exchange. In the OSI model, these activities are discussed mostly around Layers **7**, **6**, and sometimes **5**; the four-layer TCP/IP model groups them under **Application**.
2. **Transport:** **TCP** establishes and maintains an end-to-end byte-stream connection and provides reliable, ordered delivery. Both models call this **Transport** (OSI **4**).
3. **Internet / Network:** **IP** carries packets with source and destination IP addresses across interconnected networks. TCP/IP calls this **Internet**; OSI calls the corresponding responsibility **Network** (Layer **3**).
4. **Link / Data Link + Physical:** The host sends each IP packet over a local link, such as Ethernet or Wi-Fi, using link-specific framing and physical signals. TCP/IP groups this as **Link**; OSI separates **Data Link** (Layer **2**) and **Physical** (Layer **1**).
5. **At the destination:** The receiving system processes the information through the corresponding functions to make the request available to the server application.

**Important exception:** Modern HTTP/3 commonly uses **QUIC over UDP**, not HTTP over TCP. The models still help organize the functions, but don't assume every web connection uses TCP.

### Encapsulation: the labels change, not the underlying journey

For the TCP example, a useful simplified view is:

```text
HTTP request data
    ↓ TCP adds transport information
TCP segment
    ↓ IP adds network addressing information
IP packet
    ↓ Ethernet adds local framing information
Ethernet frame
    ↓ transmitted as signals over the medium
Bits / signals
```

On the receiving side, the process is reversed. The models organize how we **describe** these responsibilities; they do not change the packet just because we choose a different diagram.

## 4. Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| **Reference model** | Conceptual framework for organizing functions | OSI's seven layers |
| **Protocol suite** | Collection of protocols designed to work together | TCP/IP includes IP, TCP, UDP and more |
| **Abstraction** | Leaving out detail to focus on useful relationships | Four groups instead of seven |
| **Mapping** | Relating categories from one model to another | OSI 3 ↔ TCP/IP Internet |
| **Application layer** | Services and formats used by networked applications | HTTP and DNS |
| **Transport layer** | End-to-end transport functions | TCP and UDP |
| **Internet layer** | IP-based addressing and forwarding between networks | IPv4, IPv6 |
| **Link layer** | Communication over a local network link | Ethernet, Wi-Fi |
| **Encapsulation** | Adding protocol information around data for transmission | TCP → IP → Ethernet |
| **PDU** | Protocol data unit: the unit handled at a given layer | Segment, packet, frame |

## 5. Protocol classification lab: which map are you using?

Use this table as a reference, then cover the last two columns and try to reconstruct them.

| Protocol / technology | OSI classification (typical) | Four-layer TCP/IP classification | Why |
|---|---|---|---|
| HTTP | 7 — Application | Application | Web application protocol |
| DNS | 7 — Application | Application | Name-resolution protocol |
| SMTP | 7 — Application | Application | Email transfer protocol |
| TCP | 4 — Transport | Transport | Reliable ordered byte stream |
| UDP | 4 — Transport | Transport | Datagram transport without built-in delivery guarantees |
| IPv4 / IPv6 | 3 — Network | Internet | Logical addressing and routing |
| ICMP | 3 — Network | Internet | Network control and error messages |
| Ethernet | 2 — Data Link; 1 for physical signaling | Link | Local framing and media specifications |
| Wi-Fi (IEEE 802.11) | 2 — Data Link; 1 — Physical | Link | Wireless MAC and physical transmission |
| TLS | Often associated with 6 conceptually; not a clean one-layer fit | Usually treated within the Application grouping | Protects application communications; sits above transport in common stacks |
| ARP (IPv4) | Often described at the Layer 2/3 boundary | Often treated as link-adjacent | Resolves IPv4 next-hop addresses to link-layer addresses on a local network |

**Classification rule:** When a protocol crosses conceptual boundaries, explain **what it does** instead of forcing a misleading one-number answer.

## 6. Something has gone terribly wrong: use both maps

**Scenario:** A browser cannot load an internal website. A technician reports:

- The device has an IP address.
- The hostname resolves to the expected server IP.
- The server responds to a TCP connection attempt on port 443.
- The browser reports a certificate validation error.

**Reasoning:** IP addressing, DNS resolution, and TCP connectivity appear to be functioning for the tested path. The next investigation should focus on the **TLS certificate validation process**: trust chain, hostname match, validity period, or local trust configuration.

- **TCP/IP view:** Investigate functions commonly grouped into the **Application** layer, above the working TCP connection.
- **OSI view:** Certificate presentation and cryptographic representation are often discussed using **Presentation-layer concepts**, while the browser's handling of the error is an **Application-layer** concern.

**Defensible conclusion:** The evidence points toward a TLS trust or validation issue, not a proven cabling or routing failure. It does **not** prove the entire network is healthy.

**Why both models help:** The TCP/IP map quickly narrows the broad functional area. The OSI map offers a finer vocabulary for describing the kind of problem.

## 7. Common misconceptions worth retiring

| Myth | More accurate explanation |
|---|---|
| “TCP/IP is just OSI with three layers deleted.” | The models have different origins and purposes; the common mapping groups related functions. |
| “Every OSI layer must exist as a separate piece of software.” | OSI layers are conceptual responsibilities, not a requirement for seven distinct modules. |
| “The TCP/IP model always has exactly four layers.” | The **four-layer** model is common; some teaching materials use a **five-layer** model that separates Data Link and Physical. |
| “TLS is always exactly Layer 6.” | That's a useful conceptual association, not a universal layer assignment in deployed stacks. |
| “TCP/IP means every application uses TCP.” | TCP/IP names a suite; applications may use UDP, including QUIC-based HTTP/3. |

## 8. Check your understanding

Try these without looking at the mapping table.

1. **Recall:** Name the four layers in the common four-layer TCP/IP model, from top to bottom.
2. **Map:** Which three OSI layers map broadly to TCP/IP's Application layer?
3. **Classify:** Place **TCP**, **IP**, **DNS**, and **Ethernet** in both models.
4. **Explain:** What does *abstraction* mean here? Why might a four-layer map be useful even when the seven-layer map has more detail?
5. **Scenario:** A network engineer sees an IP routing problem. Which TCP/IP layer and OSI layer describe the main responsibility?
6. **Scenario:** A TCP connection succeeds, but the browser rejects the server's TLS certificate. Which broad TCP/IP layer would you investigate next, and why is assigning a single OSI layer less straightforward?
7. **Reasoning challenge:** A classmate says, “HTTP is always carried by TCP because HTTP is an application-layer protocol.” What is missing from that statement?

<details>
<summary><strong>Show answers and explanations</strong></summary>

1. **Application → Transport → Internet → Link (Network Access).** This is the four-layer teaching convention used throughout this guide.
2. **OSI Layers 7, 6, and 5.** Application, Presentation, and Session functions are grouped broadly under TCP/IP Application.
3. **TCP:** OSI 4 / TCP/IP Transport. **IP:** OSI 3 / TCP/IP Internet. **DNS:** OSI 7 / TCP/IP Application. **Ethernet:** OSI 2 and physical specifications at 1 / TCP/IP Link. The classification follows each technology's primary responsibility.
4. **Abstraction** is simplifying details to focus on the relationships that matter for a task. Four layers provide a compact working view; seven help discuss finer responsibilities.
5. **TCP/IP Internet; OSI Layer 3 (Network).** IP routing is concerned with forwarding packets between networks.
6. **TCP/IP Application** is the broad area. In OSI language, TLS is often associated with Presentation functions, but the browser's certificate checks and error handling also involve application behavior; there isn't one universally mandated layer label.
7. **HTTP/3 uses QUIC over UDP.** An application-layer protocol doesn't dictate one universal transport for every version or implementation.

</details>

## 9. Teach it back in 60 seconds

Without looking at the diagram, explain to a colleague:

- Why the OSI and TCP/IP models are like two maps of the same city.
- How the **top three** and **bottom two** OSI layers map to the four-layer TCP/IP view.
- Where **HTTP**, **TCP**, **IP**, and **Ethernet** fit.
- One reason a protocol might not fit perfectly into a single OSI layer.

If you can explain the **reason for the mapping**, not just recite the numbers, you've learned the useful part.

## 10. Five takeaways to remember tomorrow

1. **OSI is a seven-layer conceptual reference model; TCP/IP is a real protocol suite often taught using four layers.**
2. **TCP/IP Application broadly groups OSI 7–5; TCP/IP Link broadly groups OSI 2–1.**
3. **Transport ↔ OSI 4, and Internet ↔ OSI 3** are the simplest one-to-one mappings.
4. **A model is a map, not the territory.** Some technologies cross conceptual boundaries.
5. **Use the model that helps answer the question**—and translate between them when communicating or troubleshooting.

## Further learning

**Primary technical references:**

- [RFC 1122 — Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122) — internet architecture and host communication layers.
- [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293) — TCP behavior and responsibilities.
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768) — UDP's core specification.
- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791) — foundational IPv4 specification.

**Optional practice:** Open a packet capture from a system you own or an authorized training lab. Identify an Ethernet frame, an IP packet, and a TCP segment or UDP datagram, then describe each using **both** models. You don't need to memorize every header field to complete the exercise.

**Next guide:** [03 — TCP vs. UDP: Reliability vs. Speed](03-tcp-vs-udp.md)

---

© 2026 GhostParticle404.
