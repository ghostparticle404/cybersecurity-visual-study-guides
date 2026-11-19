![Guide 14: Cookies, Sessions & Tokens — coat-check tickets, VIP passes, and browser security attributes](../00-visual-guides/02.14-cookies-sessions-tokens.png)

# 14 — Cookies, Sessions & Tokens

**Cybersecurity Visual Study Guides · 02 — Web & Internet Security**  
**Estimated learning time:** 25–30 minutes, including practice

How does a website remember that you logged in when HTTP requests are separate messages? Cookies, sessions, and tokens are three related—but different—parts of the answer. This lesson shows what each does, how they fit together, and which security settings matter.

## Why this matters

You sign in to a website, open another page, and remain signed in. Behind the scenes, the browser sends information that lets the application recognize an ongoing login. Understanding that mechanism helps you investigate account hijacking, review application security, and troubleshoot authentication problems without confusing a browser storage mechanism with an authentication credential.

## Learning objectives

By the end, you should be able to:

1. **Distinguish** a cookie, a server-side session, a session identifier, and a bearer token.
2. **Trace** a typical login and subsequent authenticated HTTP request.
3. **Interpret** `Set-Cookie`, `Cookie`, `Secure`, `HttpOnly`, and `SameSite` in an example.
4. **Explain** common session and token threats and the limits of cookie protections.
5. **Inspect** cookie attributes in browser developer tools without exposing credentials.

## The big picture, in plain English

**HTTP is stateless:** one request does not automatically carry the context of earlier requests. Websites therefore need a way to associate requests with a returning visitor.

- A **cookie** is a small piece of data the browser stores for a site and may automatically attach to matching HTTP requests. Cookies can store preferences or credentials such as a session ID.
- A **session** is application-maintained state associated with a user's interactions. In a common *server-side session* design, the server stores session data and the browser holds an opaque session ID.
- A **token** is a credential or data object presented to a service. A **bearer token** grants access to whoever possesses a valid token, subject to the service's authorization checks. Tokens can be opaque or structured (for example, JSON Web Tokens, or JWTs), and can be stored or transmitted in several ways—including cookies.

**These are not three mutually exclusive login systems.** A cookie can carry a session ID *or* a bearer token; a server-side session can coexist with API tokens.

## A memorable story: coat-check tickets and VIP passes

Imagine arriving at a busy conference. You hand your coat to an attendant and receive ticket **#0427**. The ticket does not contain your coat: it is a reference that lets the attendant retrieve the correct item from the coat room. A server-side session ID works similarly: the browser presents an identifier, and the server looks up associated state.

Later, a VIP organizer hands you a pass that authorizes entry to a particular lounge. A bearer token is like that pass: a gatekeeper may accept it as evidence that the holder has permission to access something. Losing the pass could allow someone else to use it until it expires or is revoked.

The **cookie** is not necessarily either ticket or pass. Think of it as the envelope or pocket your browser uses to hold a credential and automatically present it to the right website.

### Where the analogy stops working

A session ID is not proof of someone's physical identity; it is a credential that must be unpredictable and protected. Not all sessions are stored entirely on servers, and not all tokens are VIP-style passes or JWTs. Browsers apply domain, path, security, and cross-site rules to cookies; conference staff do not. A stolen bearer token may be replayed remotely, with no physical pass needed.

## Translate the story into technical reality

| Story element | Real technical concept | Why it matters |
|---|---|---|
| Coat-check record | Server-side session state | The application retains login-related state. |
| Ticket number | Opaque session identifier | The browser supplies a lookup key; the server verifies it. |
| VIP pass | Bearer access token | Possession of a valid credential may authorize a request. |
| Pocket holding the ticket | Browser cookie storage | The browser stores a value and sends it when matching rules permit. |
| Checkpoint attendant | Authentication/authorization logic | The service decides whether a credential is valid and sufficient. |
| Pass expiration | Session/token expiration | Limits the period a credential remains usable. |

## How it actually works — step by step

### Example A: a server-side session stored in a cookie

**1. The user signs in.** The browser submits credentials over HTTPS to the application.

**2. The server verifies the login.** If successful, it creates session state, associates it with a cryptographically unpredictable session identifier, and sends a response such as:

```http
HTTP/1.1 200 OK
Set-Cookie: __Host-session=RANDOM_OPAQUE_VALUE; Path=/; Secure; HttpOnly; SameSite=Lax
Content-Type: text/html
```

This is an **illustrative value**, not a real session credential. The `__Host-` cookie-name prefix adds browser-enforced constraints: `Secure`, `Path=/`, and no `Domain` attribute.

**3. The browser stores the cookie.** On later matching requests, it may send:

```http
GET /account HTTP/1.1
Host: example.com
Cookie: __Host-session=RANDOM_OPAQUE_VALUE
```

**4. The server checks the session.** It looks up the identifier, verifies validity and expiration, and authorizes the requested action. A valid login does not automatically authorize every resource.

**5. The session ends.** Logout should invalidate the server-side session; deleting the browser cookie alone does not guarantee that a copied identifier becomes unusable. Idle and absolute timeouts can also end a session.

### Example B: a bearer token sent to an API

After authentication, an application may obtain an access token and send it explicitly:

```http
GET /api/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer EXAMPLE_ACCESS_TOKEN
```

The API checks the token's validity, expiry, intended audience, and permissions as applicable. An opaque token may require server-side lookup or introspection; a signed JWT may be validated locally using an appropriate key and strict algorithm/claim checks. **A JWT is a format, not a guarantee of security or proof that a user is authorized for every action.**

### Cookies, sessions, and tokens compared

| Question | Cookie | Server-side session | Bearer token |
|---|---|---|---|
| What is it? | Browser storage and HTTP transmission mechanism | Application state associated with interactions | Credential accepted from its holder |
| Where is it? | Browser; transmitted to matching site | Primarily server-side in this design | Wherever the application stores it; presented to service |
| What might it contain? | Preference, session ID, or token | User ID, login state, other server-side data | Opaque value or structured claims |
| How is it sent? | Automatically on qualifying HTTP requests | Usually referenced by an identifier | Often `Authorization: Bearer …`; sometimes cookie |
| Main risk | Exposure or unintended cross-site sending | Session fixation/hijacking or poor invalidation | Theft and replay until invalidated/expired |

### Cookie attributes you must recognize

| Attribute | What it does | Important limit |
|---|---|---|
| `Secure` | Restricts sending the cookie to secure connections (typically HTTPS). | Does not encrypt the cookie *at rest* or prevent XSS. |
| `HttpOnly` | Prevents JavaScript from reading the cookie via `document.cookie`. | The browser still sends it on qualifying requests; XSS may still perform actions as the user. |
| `SameSite=Strict` | Restricts cross-site cookie sending more strongly. | May disrupt legitimate cross-site navigation flows. |
| `SameSite=Lax` | Restricts many cross-site requests while allowing some top-level navigations. | Not a complete cross-site request forgery (CSRF) defense. |
| `SameSite=None` | Allows cross-site cookie sending where browser policies permit. | Requires `Secure` in modern browsers; third-party cookie policies may still block it. |
| `Path` / `Domain` | Control which request URLs receive the cookie. | These are scoping rules, not full security boundaries. |
| `Expires` / `Max-Age` | Set browser-side cookie lifetime. | Does not itself revoke a server-side session or stolen token. |

**Important:** `SameSite=None` without `Secure` is generally rejected by modern browsers; it is not inherently equivalent to a CSRF vulnerability. Actual exposure depends on the application, browser behavior, and other defenses.

### What about session cookies versus persistent cookies?

A **session cookie** lacks an explicit `Expires` or `Max-Age` and is ordinarily scoped to a browser session, although session restoration can affect when it disappears. A **persistent cookie** has an explicit lifetime. Neither label tells you whether the server-side login session remains valid.

## Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| `Set-Cookie` | Response header instructing a browser to store a cookie | `Set-Cookie: theme=dark; Secure` |
| `Cookie` | Request header carrying matching cookies | `Cookie: theme=dark` |
| Session ID | Identifier referencing an application's session | Should be unpredictable and regenerated after authentication where appropriate |
| Bearer token | Credential usable by its holder | `Authorization: Bearer …` |
| JWT | JSON Web Token, a structured token format | May carry signed claims; not automatically encrypted |
| CSRF | Cross-Site Request Forgery | Tricks a browser into sending an unwanted authenticated request |
| XSS | Cross-Site Scripting | Injected script may act in a site's origin or steal accessible tokens |
| `HttpOnly` | Blocks JavaScript reading a cookie | Helps reduce direct cookie theft via XSS |
| `SameSite` | Controls some cross-site cookie sending | `Strict`, `Lax`, `None` |
| Session fixation | Attacker causes use of a known session identifier | Rotate the identifier after login |

## A realistic cybersecurity application: review a login cookie

**Situation:** A security analyst reviews a fictional office portal after reports of suspicious account activity. In the browser's cookie panel, the analyst observes a session cookie with `Secure` enabled, `SameSite=Lax`, and `HttpOnly` disabled.

**Evidence:** The session cookie is accessible to page JavaScript, although HTTPS protects it during transit and `SameSite=Lax` restricts some cross-site requests.

**Reasoning:** Missing `HttpOnly` increases the risk that an XSS vulnerability could expose the cookie. This observation **does not prove** XSS exists or that an attacker stole the cookie. `SameSite=Lax` does not remove the need for CSRF analysis, and server-side expiration and invalidation still need review.

**Defensible conclusion:** Recommend `HttpOnly` for a session identifier unless the application has a justified need for script access; verify `Secure`, appropriate `SameSite`, session rotation, timeout, logout invalidation, and CSRF protections. Investigate suspicious logins using authorized application and identity-provider logs rather than copying live session values.

## Practice: inspect cookie security attributes

Use a website you own, an authorized training application, or a public site where you are simply viewing your **own** browser's cookies. Never paste live cookie values or access tokens into public reports.

1. Open the site using **HTTPS**.
2. Open Developer Tools (often **F12** or **Inspect**).
3. In Chrome/Chromium, find **Application → Storage → Cookies**; in Firefox, find **Storage → Cookies**. Menus vary by version.
4. Select the site and examine the **cookie name**, **domain**, **path**, **expiration**, **Secure**, **HttpOnly**, and **SameSite** fields.
5. Open the **Network** panel, select a request, and inspect the request's cookie information and any response `Set-Cookie` headers. Sensitive cookie values may be hidden by browser tooling.
6. Record only non-secret attributes and explain whether they fit the cookie's purpose.

**Reflection:** If a cookie is called `theme`, would you expect the same protections as a session identifier? Why might the answer differ?

## Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “A cookie is a session.” | A cookie can store a session ID, but a session is application state or continuity. |
| “All tokens are JWTs.” | Bearer tokens can be opaque; JWT is one token format. |
| “Sessions always live on the server.” | Server-side sessions are common, but applications also use client-side session mechanisms. |
| “`HttpOnly` stops XSS.” | It limits JavaScript access to a cookie; injected scripts can still cause damage. |
| “`SameSite` completely prevents CSRF.” | It reduces some cross-site cookie sending; defense in depth remains necessary. |
| “Logging out by deleting a cookie revokes the credential.” | Server-side invalidation or token revocation/expiry is needed to stop reuse of a copied credential. |

## Check your understanding

1. **Define:** What is the difference between a cookie and a server-side session?
2. **Identify:** Which HTTP response header tells the browser to store a cookie?
3. **Interpret:** What does `HttpOnly` protect against, and what does it *not* prevent?
4. **Scenario:** A session cookie has `Secure` and `SameSite=Lax`, but not `HttpOnly`. What specific risk increases? Does that prove an attack occurred?
5. **Scenario:** An attacker copies a valid bearer token from an insecurely stored file. Why might the attacker be able to use it without knowing the account password? Name two mitigations.
6. **Compare:** What is the difference between an opaque session ID and a signed JWT? Does either format alone guarantee authorization?
7. **Reason it through:** A user logs out, but a previously copied session ID still works. What likely went wrong, and what should the application do?

## Answer key with explanations

<details>
<summary>Show answers and explanations</summary>

**1.** A cookie is browser-stored data sent under matching HTTP rules. A server-side session is application state maintained by the server; a cookie often carries the session ID that references it.

**2.** `Set-Cookie`. It appears in an HTTP **response**; the browser later sends applicable values in a `Cookie` **request** header.

**3.** `HttpOnly` prevents ordinary page JavaScript from reading the cookie through `document.cookie`, which helps reduce direct theft via XSS. It does not prevent all XSS actions, automatically stop CSRF, or protect other tokens stored in script-accessible locations.

**4.** Direct cookie theft through an XSS vulnerability becomes easier if script can read the session cookie. Missing `HttpOnly` is a configuration weakness, **not evidence** that an XSS vulnerability or compromise occurred.

**5.** A bearer token is accepted based on possession and validity, not a password re-entry for every request. Mitigations include short token lifetimes, secure storage, revocation/rotation where supported, least-privilege scopes, and sender-constrained tokens where appropriate.

**6.** An opaque session ID is a reference whose meaning is typically checked server-side. A signed JWT contains structured claims and can be cryptographically verified. Both still require correct validation, expiration checks, and authorization decisions.

**7.** The server may have removed the browser's cookie without invalidating the server-side session, or failed to revoke the relevant credential. Logout should invalidate the session at the server and expire the browser cookie; sensitive applications should also manage concurrent sessions and token revocation according to their threat model.

</details>

## Teach it back in 60 seconds

Without looking at the lesson, explain:

1. Why a browser cookie and a server-side session are not the same thing.
2. How a session ID travels from server to browser and back.
3. What `Secure`, `HttpOnly`, and `SameSite` each change.
4. Why a stolen bearer token can be dangerous even when the account password remains secret.

## Key takeaways

1. **Cookies transport or store data; sessions preserve application context; tokens act as credentials.** They can be used together.
2. **A session ID is a secret.** Make it unpredictable, protect it in transit and storage, rotate it when appropriate, and invalidate it on logout.
3. **`Secure`, `HttpOnly`, and `SameSite` solve different problems.** None makes an application invulnerable.
4. **Bearer means possession matters.** Treat access tokens like passwords, with suitable expiry, scope, and revocation controls.
5. **Inspect attributes, not secrets.** Browser Developer Tools can reveal configuration risks without disclosing live credentials.

## Further learning and references

**Primary documentation**

- [RFC 6265 — HTTP State Management Mechanism](https://www.rfc-editor.org/rfc/rfc6265) — foundational cookie semantics.
- [MDN — Using HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies) — browser behavior and attributes.
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) — session ID and cookie security practices.
- [OWASP Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) — CSRF controls and `SameSite` limitations.
- [RFC 6750 — OAuth 2.0 Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750) — bearer tokens in HTTP.
- [RFC 7519 — JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519) — JWT format and claims.

**Related lesson:** [13 — HTTP Requests & Responses](13-http-requests-responses.md), for the headers that carry cookies and tokens.

---

© 2026 GhostParticle404.
