![REST APIs: Endpoints, Requests & Authentication](../00-visual-guides/02.16-rest-apis.png)

# 16 — REST APIs: Endpoints, Requests & Authentication

**Series:** Cybersecurity Visual Study Guides · **Category:** 02 — Web & Internet Security  
**Estimated time:** 25–30 minutes, including practice

A website can display information for a person, while an API makes information and actions available to software. This lesson teaches you to read a basic REST-style API exchange, recognize authentication and authorization decisions, and investigate a request without confusing a successful HTTP response with secure access control.

## 1. Why this matters

A mobile app displays your account details. A monitoring dashboard pulls alerts from a service. An online store checks inventory. These systems often exchange information through APIs. For defenders and testers, understanding the method, endpoint, headers, response, and permissions helps explain what a system is doing—and whether it is doing more than it should.

## 2. Learning objectives

By the end, you should be able to:

1. **Identify** a resource, endpoint, HTTP method, and JSON representation in an API exchange.
2. **Interpret** request headers, a response status, and a response body.
3. **Compare** API keys, bearer tokens, and OAuth 2.0 at an introductory level.
4. **Distinguish** authentication from authorization using a concrete example.
5. **Inspect** a sample request and **reason** about an access-control failure.

## 3. The big picture, in plain English

An **application programming interface (API)** is a defined way for software components to interact. A **REST** (Representational State Transfer) API generally organizes interactions around **resources** identified by URLs and uses HTTP methods and representations to act on them. REST is an architectural style, not a single wire protocol or a guarantee that every API follows all REST constraints.

An **endpoint** is a URL where an API operation is available. A **resource** is the thing the API represents—such as a book, account, or alert. **JSON** (JavaScript Object Notation) is a common data format for request and response bodies, but APIs can use other formats.

## 4. A memorable story: the vending machine with an instruction manual

Imagine a vending machine with labeled slots and a detailed manual. The slot number identifies what you want; the button you press tells the machine what to do. You may need a valid payment card or membership pass before the machine will dispense an item. The display then reports whether your request succeeded, failed, or requires another step.

An API is similar: the **endpoint** identifies a resource, the **HTTP method** describes the requested operation, **credentials** may establish who or what is making the request, and the **response** reports the outcome.

### Where the analogy stops working

Real APIs do not always map one button to one action. `POST` can initiate operations beyond resource creation; authentication does not guarantee authorization; and a valid token may have a narrow scope. A machine's slot labels also cannot illustrate every REST constraint, caching rule, or HTTP behavior.

## 5. Translate the story into technical reality

| Story element | Technical concept | Why it matters |
|---|---|---|
| Item or slot | Resource and endpoint, such as `/books/42` | Identifies what the request concerns |
| Button or instruction | HTTP method, such as `GET` | Expresses the requested operation |
| Membership pass | API key, access token, or other credential | May identify or authenticate the caller |
| Permission to access a slot | Authorization policy | Determines whether the caller may perform this action on this resource |
| Machine display | HTTP status and response body | Communicates the result |
| Product details | JSON representation | Carries structured data |

## 6. How it actually works — step by step

We'll use an **illustrative** books API hosted at `api.example.com`. These are sample messages, not output from a live service.

### Step A: Choose a resource and method

`GET /books/42` asks for the representation of book 42. The path `/books/42` identifies the resource; `GET` requests retrieval. The complete HTTPS URL would be `https://api.example.com/books/42`.

### Step B: Send an HTTP request

```http
GET /books/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer EXAMPLE_REDACTED_TOKEN
```

- **Request line:** `GET` + `/books/42` + HTTP version (shown in HTTP/1.1 text format).
- **Host:** identifies the requested host.
- **Accept:** expresses the preferred response representation.
- **Authorization:** supplies a bearer credential in this example. **Never put a real token in public screenshots or repositories.**

A `GET` request usually has no body. Other requests, such as `POST` and `PUT`, often carry JSON.

### Step C: The server authenticates and authorizes

The API validates the credential according to its configuration (for example, signature, issuer, audience, expiration, or lookup). It then checks whether the caller is allowed to read **this particular book**. Authentication asks **who or what is calling?** Authorization asks **is this operation permitted?** A valid token is not blanket permission.

### Step D: Read the response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "title": "Network Security",
  "author": "Example Author"
}
```

The status `200 OK` indicates the request was handled successfully at the HTTP level. `Content-Type` identifies the response media type; the body is a JSON object. The status alone does not establish that the server enforced access control correctly.

### Step E: Compare common methods

| Method | Common use | Example | Important nuance |
|---|---|---|---|
| `GET` | Retrieve a representation | `GET /books/42` | Defined as safe; should not request state changes |
| `POST` | Submit data or request processing | `POST /books` | Not limited to creating resources |
| `PUT` | Create or replace target resource state | `PUT /books/42` | Defined as idempotent |
| `PATCH` | Apply a partial modification | `PATCH /books/42` | Idempotency depends on the patch semantics |
| `DELETE` | Request resource removal | `DELETE /books/42` | Defined as idempotent; response need not be identical each time |

**Idempotent** means repeating the same intended request has the same intended effect on server state as making it once—not that the response must always have the same status or body.

### Step F: Interpret common status codes

| Code | Name | Typical meaning |
|---|---|---|
| `200` | OK | Successful request |
| `201` | Created | A resource was created |
| `204` | No Content | Successful response without a body |
| `400` | Bad Request | Server cannot or will not process the request due to a client-side issue |
| `401` | Unauthorized | Authentication credentials are missing or invalid for the requested resource |
| `403` | Forbidden | Server understood the request but refuses to fulfill it |
| `404` | Not Found | Resource not found, or server does not disclose its existence |
| `429` | Too Many Requests | Request rate limit reached |
| `500` | Internal Server Error | Unexpected server-side failure |

The HTTP code tells you the **reported result**. The response body and surrounding evidence help explain why.

## 7. Essential terminology

| Term | Plain-English meaning | Example or implication |
|---|---|---|
| API | Defined interface for software interaction | Mobile app calls an inventory service |
| REST | Architectural style for resource-oriented systems | Resource URLs plus HTTP semantics |
| Endpoint | URL for an API operation | `/books/42` |
| Resource | Entity represented by an API | A book or customer account |
| JSON | Structured text data format | `{"id":42}` |
| Header | Metadata associated with HTTP messages | `Accept: application/json` |
| API key | Credential often used to identify an application or project | Must be protected and restricted |
| Bearer token | Credential usable by whoever possesses it | Theft may enable unauthorized calls |
| OAuth 2.0 | Authorization framework for delegated or controlled access | Access token with specific scopes |
| JWT | JSON Web Token: compact claims format | Not every bearer token is a JWT |
| Authentication | Verifying identity or credential | Validate an access token |
| Authorization | Deciding allowed operations | Can user A read user B's data? |
| Scope | Declared limit on token permissions | `books:read` |
| Status code | HTTP result indicator | `403 Forbidden` |

## 8. Authentication patterns: similar goals, different mechanisms

**API keys** are typically issued to an application or project. Their meaning varies by service: some identify a caller, some grant access, and some are used for metering. An API key is not automatically equivalent to user authentication.

**Bearer tokens** are presented to a resource server, often with `Authorization: Bearer ...`. Whoever possesses a valid bearer token may be able to use it, subject to the token's scope, audience, validity, and server policy. Keep tokens out of logs, source code, and URLs.

**OAuth 2.0** is an authorization framework that can issue access tokens for defined access. OAuth itself is **not a login protocol**; OpenID Connect adds an identity layer commonly used for login. A JWT is a token **format**, not a synonym for OAuth or a guarantee of security.

| Pattern | What it commonly conveys | Key risk |
|---|---|---|
| API key | Application/project identity or service access | Exposed or overprivileged key |
| Bearer access token | Authority to make certain requests | Stolen token replay |
| OAuth 2.0 | Framework for granting controlled access | Incorrect scope, audience, or token validation |

## 9. Real-world cybersecurity application: a book that belongs to someone else

**Situation:** An application uses `GET /accounts/123/orders/42` to show an order. A logged-in user changes `42` to `43` in an authorized training lab. The server returns `200` and details for another user's order.

**Evidence:** The two requests use the same authenticated session, but the second response contains an order owned by a different account. Ownership should be verified against the lab's ground truth; a changed ID alone does not prove a vulnerability.

**Reasoning:** The API may have validated the caller's identity but failed to check whether that caller is authorized to access the specific object. This is the pattern known as **Broken Object Level Authorization (BOLA)** in OWASP API Security guidance.

**Defensible conclusion:** If ownership and lack of permission are confirmed, this is an object-level authorization failure. The fix is server-side authorization for **every** protected object access, not merely hiding identifiers in the interface.

**Safe practice:** Reproduce this only in a lab or system where you have explicit permission. Do not enumerate other people's records on live services.

## 10. Common misconceptions and pitfalls

| Myth | More accurate explanation |
|---|---|
| “An API is automatically RESTful because it uses HTTP.” | REST has architectural constraints; many HTTP APIs follow only some conventions. |
| “A valid token means the caller can access any resource.” | Authorization must check the operation and target resource. |
| “OAuth is the same thing as JWT.” | OAuth is a framework; JWT is one possible token format. |
| “A `200` response proves the request was secure.” | It reports HTTP-level success, not correct permissions or safe data handling. |
| “All `POST` requests create resources.” | `POST` can request many kinds of processing. |

## 11. Check your understanding

1. In `GET /books/42`, which part is the method and which is the resource path?
2. What does the `Accept: application/json` header communicate?
3. Explain the difference between authentication and authorization using an API example.
4. **Scenario:** A client receives `401` after omitting its access token. What should you investigate first?
5. **Scenario:** A validly authenticated user retrieves another user's private order by changing an ID. What security control may be missing? Explain your reasoning.
6. Is every bearer token a JWT? Is OAuth 2.0 the same as a login protocol?
7. **Scenario:** An API responds `204 No Content` after a `DELETE`. What does the response tell you, and what does it *not* prove about authorization?

## 12. Answer key with explanations

<details>
<summary>Show answers and explanations</summary>

**1.** `GET` is the HTTP method; `/books/42` is the resource path. The host is supplied separately in the URL or request authority.

**2.** The client prefers a JSON response representation. It does not guarantee the server will return JSON; inspect the actual `Content-Type`.

**3.** Authentication verifies a caller or credential; authorization checks whether that caller may read a specific book or modify a specific account. A valid login can coexist with a denied operation.

**4.** Check whether the endpoint requires credentials and whether the `Authorization` header is missing or malformed. A `401` often includes a `WWW-Authenticate` challenge; do not assume an expired token when none was sent.

**5.** Object-level authorization may be missing. The server must check that the caller has permission for the *specific order*, not just that the caller is logged in. Confirm the ownership and policy before concluding there is a flaw.

**6.** No. Bearer describes how possession of a credential grants access; JWT is one possible format. OAuth 2.0 is an authorization framework; OpenID Connect is commonly used for identity/login.

**7.** `204` reports successful handling without a response body. It does not prove the server checked whether the caller had permission to delete the resource.

</details>

## 13. Teach it back in 60 seconds

Without looking at the lesson, explain:

1. How a method, endpoint, headers, and body fit together in an API request.
2. Why an access token and an authorization check are not interchangeable.
3. What you would inspect first when an API returns `401`, `403`, or `404`.

## 14. Key takeaways

- **Resources and endpoints** identify what an API exposes; **HTTP methods** describe the requested action.
- **Headers and JSON bodies** carry metadata and structured information; **status codes** communicate the reported outcome.
- **Authentication** identifies a caller; **authorization** decides whether that caller can perform this action on this resource.
- **Bearer tokens** must be protected because possession can confer access; OAuth, JWT, and API keys are different concepts.
- **Secure API design** requires server-side permission checks and careful interpretation of requests and responses.

## 15. Further learning and references

Primary and authoritative reading:

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110) — methods, status codes, and HTTP message semantics.
- [RFC 8259 — The JavaScript Object Notation (JSON) Data Interchange Format](https://www.rfc-editor.org/rfc/rfc8259) — JSON syntax and data model.
- [RFC 6750 — OAuth 2.0 Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750) — bearer-token transmission and security considerations.
- [RFC 6749 — The OAuth 2.0 Authorization Framework](https://www.rfc-editor.org/rfc/rfc6749) — authorization framework concepts.
- [OWASP API Security Top 10 (2023)](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) — API security risks including broken object-level authorization.

**Optional practice:** In an API lab you own or are authorized to use, inspect a `GET` request, record the method/path/headers/status/response body, and explain which observations relate to authentication versus authorization. Redact all secrets before sharing screenshots.

**Related guides:** [13 — HTTP Requests & Responses](13-http-requests-responses.md) · [14 — Cookies, Sessions & Tokens](14-cookies-sessions-tokens.md) · [15 — What Happens When You Visit a Website?](15-visiting-a-website.md)

---

© 2026 GhostParticle404.
