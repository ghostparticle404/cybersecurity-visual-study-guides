![DNS lookup visual study guide showing recursive resolution, DNS records, and troubleshooting](../00-visual-guides/01.06.DNS%20Lookup%20Visual%20Study%20Guide.png)

# 06 — DNS: What Happens When You Type a URL?

**Series:** Cybersecurity Visual Study Guides  
**Category:** 01 — Networking Fundamentals  
**Estimated time:** 20–30 minutes, including practice

You enter a web address and a page appears. Somewhere between those two events, your device needs to discover where to send its network traffic. This lesson traces how the Domain Name System (DNS) helps, what different DNS records mean, and how to inspect a lookup yourself.

## Why this matters

Suppose `https://www.example.com/help` fails to load. Is the website down, is DNS returning an unexpected address, or is the connection failing *after* name resolution? Understanding DNS lets you separate those possibilities instead of diagnosing every browser error as “the internet is broken.” DNS activity also provides useful leads during security investigations.

## Learning objectives

By the end, you should be able to:

1. **Identify** the hostname within a URL and explain DNS's role in resolving it.
2. **Trace** a typical cache miss through a recursive resolver, root server, top-level domain (TLD) server, and authoritative server.
3. **Interpret** basic A, AAAA, and CNAME records, and recognize MX and TXT records.
4. **Use** `dig` or `nslookup` to inspect DNS answers and explain what the output does—and does not—prove.
5. **Compare** a normal lookup with suspicious or unexpected DNS behavior.

## The big picture, in plain English

DNS is a distributed naming system. It helps applications discover information associated with names, including the IPv4 or IPv6 addresses used to contact hosts. Humans remember names like `www.example.com`; network connections typically need destination IP addresses.

DNS does **not** resolve an entire URL. In `https://www.example.com/help`, `https` is the scheme, `www.example.com` is the hostname, and `/help` is the path. DNS can help resolve the hostname; the browser handles the remaining steps, including connecting to the server and requesting the resource.

A DNS response may also come from a cache, and modern browsers or operating systems may use different resolver arrangements. The classic sequence below is a useful model of a **cache-miss recursive lookup**, not a guarantee that every page load triggers every step.

## The memorable hook: The phone book with a gossip network

Imagine you want to call a business, but you know only its name. You ask your friend who keeps an impressively organized contact list.

“Do you have the number for Example Company?”

If your friend already has a recent entry, they hand it over. If not, they ask a directory that knows where to find listings for the right category, which points them to the office maintaining the company's official listing. Your friend writes down the answer for next time—at least until it might be out of date.

The funny part is that your friend doesn't need to know **every** number. They need to know whom to ask, how to follow referrals, and when a saved answer should expire. That is the useful part of the “gossip network”: referrals and cached information, not random rumors.

### Where the analogy stops working

DNS servers do not casually gossip or vote on the correct address. Root and TLD servers typically provide **delegation information**, while authoritative servers provide answers for the zones they serve. Recursive resolvers follow defined DNS protocols and cache responses according to applicable rules and time-to-live values. DNS answers are not automatically trustworthy just because a server supplied them; DNSSEC can authenticate certain DNS data when correctly deployed and validated.

## Translate the story into technical reality

| Story element | Real technical concept | Why it matters |
|---|---|---|
| You asking for a phone number | Client's DNS query | Starts name resolution |
| Friend checking saved contacts | Resolver cache | Can avoid repeated upstream queries |
| Friend making calls for you | Recursive resolver | Pursues the answer on the client's behalf |
| General directory | Root DNS servers | Refer the resolver toward the appropriate TLD |
| Category directory | TLD DNS servers | Refer the resolver toward authoritative nameservers |
| Official company listing | Authoritative DNS server | Provides authoritative DNS data for the relevant zone |
| Saved listing's expiration | Time to live (TTL) | Limits how long a cached record should be retained |

## How DNS actually works — step by step

Let's use a **simplified example**: you open `https://www.example.com/help`, and your device needs an IPv4 address for `www.example.com`.

### Step 1: The browser identifies the hostname

The browser parses the URL and identifies `www.example.com`. It may check relevant browser or system caches and then use the configured DNS resolution mechanism. If an answer is already cached and usable, the rest of this example may be skipped.

### Step 2: The client asks a recursive resolver

The client (or its local DNS stub resolver) sends a question such as: “What is the A record for `www.example.com`?” The recursive resolver may be provided by a network operator, an organization, or a public DNS service. Some clients use encrypted DNS transports, such as DNS over HTTPS or DNS over TLS.

### Step 3: On a cache miss, the resolver follows referrals

In a traditional iterative resolution process, the recursive resolver can:

1. Ask a **root** server where to find information for `.com`.
2. Ask a **`.com` TLD** server which authoritative nameservers serve `example.com`.
3. Ask an **authoritative nameserver** for the requested record, following any relevant CNAME alias chain as needed.

The root and TLD servers generally refer the resolver onward; they do **not** normally provide the final website address.

### Step 4: The resolver returns the answer

The resolver returns the DNS response to the client and may cache relevant data subject to TTLs and resolver policies. A response can contain an address, an alias, a negative result, or an error. A successful DNS lookup does not guarantee that the web server is reachable.

### Step 5: The browser connects

Once it has a usable destination address, the browser can establish a connection using the appropriate transport and security protocols, such as TCP and TLS or QUIC, and request `/help` using HTTP. **DNS resolution is only one part of loading the page.**

### One simplified flow

```text
Browser: https://www.example.com/help
    |
    | Need address for www.example.com
    v
Client / configured recursive resolver
    |
    +-- Cached answer? -- Yes --> Return answer
    |
    No
    v
Root server ---- referral to .com nameservers
    |
    v
.com TLD server ---- referral to example.com nameservers
    |
    v
Authoritative server ---- DNS answer (or alias/referral as applicable)
    |
    v
Recursive resolver ---- answer to client (may cache)
    |
    v
Browser ---- connection and HTTPS request
```

**Important nuance:** A resolver can obtain some information from existing caches, and a name can involve multiple DNS records, nameservers, or queries. The diagram is conceptual rather than a literal packet trace of every browser visit.

## Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| **DNS** | Domain Name System, a distributed system for naming information | Look up records for `example.com` |
| **Stub resolver** | Client-side component that sends DNS questions to a resolver | Operating system DNS client |
| **Recursive resolver** | Service that obtains an answer on the client's behalf | Organization or public resolver |
| **Authoritative nameserver** | Server providing authoritative data for a DNS zone | Nameserver for `example.com` |
| **Root server** | Entry point to the DNS hierarchy for referrals | Points toward `.com` nameservers |
| **TLD** | Top-level domain | `.com`, `.org` |
| **Cache** | Temporarily stored DNS data | Avoids repeating an upstream lookup |
| **TTL** | Time to live, the DNS record's caching lifetime | A response may be cached for its remaining TTL |
| **A** | Maps a name to an IPv4 address | `example.com` → `192.0.2.10` (illustrative) |
| **AAAA** | Maps a name to an IPv6 address | `example.com` → `2001:db8::10` (illustrative) |
| **CNAME** | Makes one DNS name an alias of another | `www.example.com` → `example.com` (illustrative) |
| **MX** | Identifies mail exchangers for a domain | Mail delivery routing |
| **TXT** | Stores text strings associated with a name | Can support SPF or domain verification |
| **NXDOMAIN** | Response code indicating that the queried name does not exist | Possible typo or nonexistent name |

The `192.0.2.0/24` IPv4 range and `2001:db8::/32` IPv6 range are reserved for documentation. Examples here are illustrative, **not live lookup results**.

### A, AAAA, and CNAME: don't mix them up

| Record | Answer type | Key distinction |
|---|---|---|
| **A** | IPv4 address | Direct address mapping |
| **AAAA** | IPv6 address | Direct address mapping |
| **CNAME** | Another DNS name | Alias; additional lookup processing may be needed to obtain an address |

A CNAME does **not** contain a port number or redirect a browser to a new URL. HTTP redirects are a separate mechanism.

## Real-world cybersecurity application: Investigate an unexpected DNS answer

**Situation:** An analyst learns that users cannot reach an internal application. The browser reports a connection error. The analyst wants to determine whether name resolution is contributing to the failure.

**Evidence to collect:**

1. Query the application's hostname with `dig` or `nslookup` from an affected system.
2. Record the answer, response code, TTL, resolver used, and time of observation.
3. Compare the result with the organization's approved DNS configuration and, where authorized, another resolver or network vantage point.
4. If DNS returns the expected address, test connectivity separately rather than declaring DNS the cause.

**Reasoning:** An `NXDOMAIN` result, an unexpected address, or inconsistent answers may indicate a typo, configuration change, stale cache, split-horizon DNS, filtering, or malicious interference. The DNS response **alone** does not prove an attack.

**Defensible conclusion:** “This system received an unexpected DNS response at the recorded time; further configuration and network evidence is needed to determine the cause.”

DNS queries can also help threat hunters identify unusual domains or suspicious query patterns, but unusual does not automatically mean malicious.

## Try it yourself: `dig` and `nslookup`

Use a domain you own or a public example intended for testing. The commands below query public DNS information; do not send private internal hostnames to public resolvers without authorization.

### Option A: `dig`

```bash
dig example.com A
```

**What it does:** Requests IPv4 A records for `example.com` using your configured DNS resolver. Inspect the status, `ANSWER SECTION`, TTL, and `SERVER` field. Output varies by resolver, network, and time.

```bash
dig example.com AAAA
```

**What it does:** Requests IPv6 AAAA records. An empty answer section does not necessarily mean DNS itself failed.

```bash
dig www.example.com CNAME
```

**What it does:** Asks specifically for CNAME records. There may be no CNAME for a particular hostname; that is normal.

```bash
dig example.com A +short
```

**What it does:** Shows a compact response, often useful for quickly checking returned addresses. For investigation, use the full output as well to preserve context.

### Option B: `nslookup`

```bash
nslookup example.com
```

**What it does:** Queries the configured resolver for address information. Output format differs across systems.

```bash
nslookup -type=MX example.com
```

**What it does:** Requests mail-exchanger records for the domain.

### Mini-investigation

Without guessing what the output *should* be, record:

- Which resolver answered your query?
- Which record type did you request?
- What answer or response status did you receive?
- Was a TTL visible, and what does it tell you?
- Does the output prove the website itself is reachable? Why or why not?

**Safety:** These are ordinary DNS queries. Keep testing within public data or environments where you have permission.

## Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “DNS translates every part of a URL.” | DNS resolves names; schemes, paths, and HTTP requests are handled separately. |
| “Every visit contacts the root DNS servers.” | Caching and resolver behavior often eliminate those queries. |
| “The root server gives you the website's IP.” | Root servers typically provide referrals to TLD nameservers. |
| “A CNAME is another IP address.” | A CNAME points to another DNS name. |
| “A correct DNS answer means the website works.” | Routing, transport, TLS, servers, and applications can still fail. |

## Check your understanding

1. In `https://www.example.com/articles/intro`, which part is the hostname that may need DNS resolution?
2. Put these roles in a typical cache-miss referral sequence: **authoritative nameserver**, **recursive resolver**, **root server**, **TLD server**.
3. What is the difference between an A record and an AAAA record?
4. A resolver has a valid cached answer. Does it necessarily query the root server again? Explain.
5. **Scenario:** `dig example.com A` returns an address, but the website still will not load. Name two possible problems *after* DNS resolution.
6. **Scenario:** A hostname returns a CNAME pointing to another name. Does that CNAME itself tell you the final IPv4 address? Explain.
7. **Investigation scenario:** Two machines return different answers for an internal hostname. List two non-malicious explanations and one piece of evidence you would collect before alleging compromise.

<details>
<summary>Show answers and explanations</summary>

1. **`www.example.com`.** The scheme (`https`) and path (`/articles/intro`) are not DNS hostnames.
2. **Recursive resolver → root server → TLD server → authoritative nameserver.** The resolver coordinates the work; upstream servers supply referrals and authoritative answers as appropriate.
3. **A = IPv4; AAAA = IPv6.** Both associate a name with an address of a different IP version.
4. **No.** A usable cached answer can be returned without repeating the full referral sequence.
5. **Examples:** network routing failure, blocked connection, unavailable server, TLS error, or application error. DNS success does not prove end-to-end connectivity.
6. **No.** A CNAME names an alias target. Address information may require following that name's A or AAAA records.
7. **Examples:** split-horizon DNS, different resolvers, different cache states, or legitimate load-balancing policies. Collect resolver identities, timestamps, full answers, and configuration context before drawing a conclusion.

</details>

## Teach it back in 60 seconds

Without looking at the diagram, explain:

1. Why DNS is needed even though applications ultimately connect to IP addresses.
2. What a recursive resolver does when its cache does not contain the answer.
3. How A, AAAA, and CNAME records differ.
4. Why a DNS answer cannot, by itself, confirm that a website is working—or that a security incident occurred.

## Key takeaways

- **DNS resolves names, not complete URLs.** The browser has additional work after resolution.
- **Recursive resolvers do the legwork.** On cache misses, they can follow referrals from root to TLD to authoritative nameservers.
- **Caches change the path.** Not every lookup involves the full DNS hierarchy.
- **A, AAAA, and CNAME answer different questions.** Address records are not the same as aliases.
- **DNS is investigative evidence, not a verdict.** Compare answers with context before diagnosing an outage or declaring malicious activity.

## Further learning and references

**Primary technical references:**

- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034)
- [RFC 1035 — Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [RFC 3596 — DNS Extensions to Support IP Version 6](https://www.rfc-editor.org/rfc/rfc3596)
- [RFC 2308 — Negative Caching of DNS Queries](https://www.rfc-editor.org/rfc/rfc2308)

**Optional practice:** Run the commands above and compare A, AAAA, CNAME, and MX queries. Capture the output and annotate what each field tells you.

**Related guide:** [05 — IP Addresses, MAC Addresses & Ports](05-ip-mac-addresses-and-ports.md)

---

© 2026 GhostParticle404.
