# HTTP — How the Web Actually Communicates

> Video-2 · From what HTTP is, to requests, responses, headers, methods, and beyond.

---

In the last video, we saw how your request travels across the internet to reach a server. But we skipped over something — once the connection is established, how does the browser actually *talk* to the server? What language do they use?

That language is **HTTP**.

---

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is a set of rules that defines how a browser and a server communicate and exchange data.

It follows a simple **client-server model**:
- The **client** (your browser) always initiates — it sends a request
- The **server** responds with whatever was asked — a webpage, an image, some data

One important rule: **the client always speaks first.** The server never randomly pushes data. It waits to be asked.

HTTP runs on top of **TCP** — the reliable protocol we covered earlier. This ensures that everything arrives completely and in order.

---

## HTTP vs HTTPS

**HTTPS** is just HTTP with an extra layer of security. The `S` stands for *Secure*.

Under the hood, HTTPS uses **TLS** (Transport Layer Security) to:
- Encrypt the data so no one in the middle can read it
- Verify the server's identity via a certificate

Everything else — the structure of requests, headers, methods — works exactly the same. HTTPS is not a different protocol, just a safer channel for HTTP to travel through.

> **SSL vs TLS:** You'll often hear both terms. SSL (Secure Sockets Layer) is the older version — it's outdated and has known vulnerabilities. TLS replaced it. Today, all "SSL certificates" actually use TLS. The old name just stuck around.

---

## HTTP is Stateless — And That's a Good Thing

HTTP is **stateless** — meaning the server doesn't remember you between requests. Every request arrives fresh, with no memory of the last one.

This sounds like a limitation, but it's actually a strength:

- **Scalable** — any server can handle any request, since none of them hold your session
- **Reliable** — if one server crashes, your next request just goes to another one
- **Simple** — no need to track who's connected to which server

> This is why login sessions use **cookies or tokens** — they're the workaround for statelessness. The client carries its own identity with every request.

---

## Anatomy of an HTTP Request

When your browser sends a request, it looks something like this:

```
GET /home HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Authorization: Bearer abc123
```

Let's break it down.

### The Request Line

```
GET /home HTTP/1.1
```

- `GET` — the **HTTP method** (what action do you want to perform?)
- `/home` — the **endpoint** (which resource are you asking for?)
- `HTTP/1.1` — the version of HTTP being used

### Headers

Everything below the request line is a **header** — key-value pairs that carry extra information about the request. After all headers, there's a blank line to signal *"headers are done."*

Think of headers like the label on a package. The label tells the post office how to handle it — fragile, urgent, return address — without opening the box.

Headers are grouped by what they describe:

---

**Request Headers** — tell the server *about the client*

| Header | What it means |
|--------|---------------|
| `User-Agent` | What's making the request — a browser, Postman, a mobile app |
| `Authorization` | Credentials — usually a Bearer token for auth |
| `Cookie` | Stored cookies being sent back to the server |
| `Accept` | What format the client wants back — `text/html`, `application/json` |

---

**General Headers** — apply to both request and response

| Header | What it means |
|--------|---------------|
| `Date` | When the request/response was created |
| `Cache-Control` | Instructions about caching — *"don't cache this"* or *"cache for 1 hour"* |
| `Connection` | Whether to keep the TCP connection alive after the request |

---

**Representation Headers** — describe the *body* of the message

| Header | What it means |
|--------|---------------|
| `Content-Type` | What format the body is in — `application/json`, `text/html` |
| `Content-Length` | How many bytes the body is |
| `Content-Encoding` | If the body is compressed — e.g. `gzip` |
| `ETag` | A unique tag for this version of a resource — used in caching |

---

**Security Headers** — protect the browser and user

| Header | What it does |
|--------|--------------|
| `Strict-Transport-Security` | Forces HTTPS — browser will never use HTTP for this site |
| `Content-Security-Policy` | Controls which scripts/resources the page is allowed to load |
| `X-Frame-Options` | Prevents your page from being embedded in an iframe (clickjacking protection) |
| `X-Content-Type-Options` | Stops the browser from guessing content type — use what the server says |
| `Set-Cookie` | Server sets a cookie on the client — can include flags like `HttpOnly`, `Secure` |

---

**Custom Headers** — developers can create their own headers for any purpose, usually prefixed with `X-`. Example: `X-Request-ID: abc-123` to trace a request through a system.

---

## HTTP Methods — What Do You Want to Do?

The **method** in a request tells the server what kind of action is being requested. It's the verb.

| Method | Intent |
|--------|--------|
| `GET` | Fetch data. No body. Just ask. |
| `POST` | Send data to create something new. Has a body. |
| `PUT` | Replace a resource completely with new data. |
| `PATCH` | Update only part of a resource. |
| `DELETE` | Remove a resource. |

**PUT vs PATCH:**
Imagine you have a user profile with name, email, and phone. `PUT` replaces the whole thing — you send the complete updated profile. `PATCH` lets you send only the fields you want to change — just the phone number, for example.

### Idempotent vs Non-Idempotent

**Idempotent** means: *call it once or call it ten times — the result is the same.*

- `GET` — idempotent. Fetching the same page 10 times doesn't change anything.
- `PUT` — idempotent. Replacing a resource with the same data 10 times still leaves it the same.
- `DELETE` — idempotent. Deleting something twice has the same result as deleting it once.
- `POST` — **not** idempotent. Submitting a payment form 10 times creates 10 charges.

This matters a lot when deciding how to design APIs and when it's safe to retry a failed request.

---

## CORS — Who's Allowed to Talk to the Server?

Imagine your frontend at `myapp.com` wants to call an API at `api.otherdomain.com`. By default, the browser **blocks this** — it's a security measure called the **same-origin policy**.

**CORS (Cross-Origin Resource Sharing)** is the mechanism that lets servers say: *"I trust these other origins — let them in."*

When the server includes a header like:
```
Access-Control-Allow-Origin: https://myapp.com
```

...the browser allows the request. Without it, you get a CORS error.

> CORS is enforced by the **browser**, not the server. If you call the same API from Postman or a backend script, you won't see any CORS errors — it's purely a browser security feature.

### Preflight Requests

For certain requests, the browser doesn't just send the request directly. It first sends a quick **OPTIONS request** to ask the server: *"Hey, will you accept this?"*

This preflight happens when:
- The method is something other than simple `GET`, `POST`, or `HEAD`
- The request includes non-standard headers like `Authorization` or custom `X-` headers
- The `Content-Type` is something other than `text/plain`, `application/x-www-form-urlencoded`, or `multipart/form-data`

> **Think of it like a building security guard.** Before letting a visitor in, the guard calls upstairs — *"Someone named Alice wants to meet Bob. Should I let her in?"* If Bob says yes, Alice enters. The preflight is that phone call. The actual request is Alice walking in.

---

## Status Codes — What Did the Server Think?

Every response comes with a **status code** — a 3-digit number that tells you how the request went.

### 1xx — Informational
Rarely seen in everyday use. Mostly used for large uploads where the server acknowledges it received part of the data.

### 2xx — Success ✓

| Code | Meaning |
|------|---------|
| `200 OK` | Standard success. Here's what you asked for. |
| `201 Created` | Something new was created — a new user, a new post. |
| `204 No Content` | Success, but nothing to return. Common for `DELETE`. |

### 3xx — Redirection

| Code | Meaning |
|------|---------|
| `301 Moved Permanently` | This URL has permanently moved. Update your bookmarks. |
| `302 Found` | Temporarily at a different URL. |
| `304 Not Modified` | Cached version is still fresh — no need to re-send. |

### 4xx — Client Error (you did something wrong)

| Code | Meaning |
|------|---------|
| `400 Bad Request` | The request is malformed or missing something. |
| `401 Unauthorized` | No valid credentials. You need to log in. |
| `403 Forbidden` | Credentials are valid, but you don't have permission. |
| `404 Not Found` | The resource doesn't exist at this URL. |
| `405 Method Not Allowed` | You used `DELETE` on an endpoint that only accepts `GET`. |
| `429 Too Many Requests` | You're sending too many requests. Slow down. |

> **401 vs 403:** 401 = *"Who are you?"* (not logged in). 403 = *"I know who you are, but no."* (logged in but not allowed).

### 5xx — Server Error (the server did something wrong)

| Code | Meaning |
|------|---------|
| `500 Internal Server Error` | Something crashed on the server. Generic catch-all. |
| `502 Bad Gateway` | The server got a bad response from an upstream service. |
| `503 Service Unavailable` | Server is down or overloaded. Try again later. |

---

## HTTP Caching — Avoid Asking for the Same Thing Twice

Every time you visit a page, should the browser re-download every image and stylesheet? That would be slow and wasteful. **Caching** solves this.

When you request a resource, the server can include headers like:
- `Cache-Control: max-age=3600` — *"Cache this for 1 hour"*
- `ETag: "abc123"` — *"This is version abc123 of the resource"*

Next time the browser asks for the same resource, it sends:
- `If-None-Match: "abc123"` — *"I have version abc123, has anything changed?"*
- `If-Modified-Since: [timestamp]` — *"Has this changed since this date?"*

If nothing changed, the server replies `304 Not Modified` — no data sent, browser uses the cached version. Fast and efficient.

---

## Content Negotiation — Speaking the Client's Language

The client can tell the server what it prefers and the server tries to match:

- **`Accept: application/json`** → client wants JSON, not HTML
- **`Accept-Language: en-US`** → client wants English
- **`Accept-Encoding: gzip`** → client can handle compressed responses

The server looks at these and responds in the best format it can. If it can't match exactly, it has fallback defaults.

---

## Persistent Connections — Don't Hang Up After Every Request

Early HTTP (1.0) opened a new TCP connection for *every single request*, then closed it. Loading a webpage with 20 images meant 20 separate TCP handshakes. Slow.

**HTTP/1.1 introduced persistent connections** — the TCP connection stays open and multiple requests can be sent through it. The `Connection: keep-alive` header enables this.

This is the default behaviour today. The connection stays open until the client or server closes it or a timeout is reached.

---

## Handling Large Data

### Multipart Requests
When uploading a form with both text and files, the request body is split into **parts** — each separated by a **boundary delimiter**.

```
Content-Type: multipart/form-data; boundary=----XYZ123

------XYZ123
Content-Disposition: form-data; name="username"

john
------XYZ123
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

[binary image data]
------XYZ123--
```

Each part has its own headers and content. The server reads until it hits the boundary.

### Chunked Transfer
For large responses (like streaming or big file downloads), the server doesn't need to know the total size upfront. It can send the data in **chunks**, each prefixed with its size. The client reassembles them in order.

Useful when: generating a large report on the fly, streaming video, real-time data.

---

## Quick Recap

```
HTTP = rules for client-server communication
HTTPS = HTTP + TLS encryption
Stateless = no memory between requests (tokens/cookies fill the gap)

Request = method + endpoint + headers + (optional body)
Response = status code + headers + body

Methods   → GET (read), POST (create), PUT (replace), PATCH (update), DELETE (remove)
Status    → 2xx (ok), 3xx (redirect), 4xx (client error), 5xx (server error)
Headers   → metadata about the request/response
CORS      → browser security — controls cross-origin access
Caching   → reuse old responses when nothing has changed
```

---

*Video-2 · System Design Notes*
