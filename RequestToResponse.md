# What Happens When You Click on example.com?

> Video-1 · A beginner-friendly walkthrough of the full request journey.

---

You open your browser, type `example.com`, and hit Enter.

Something incredible happens in the next few milliseconds — your request travels across the internet, passes through multiple layers of infrastructure, and comes back with a webpage. Let's follow that journey, step by step.

---

## Step 1 — DNS: Finding the Address

The browser doesn't understand `example.com`. Computers talk in numbers, not names. Every website lives at an **IP address** — something like `93.184.216.34`. Your browser needs that before it can do anything.

So it asks the **DNS system** (Domain Name System) — which is essentially the internet's phone book.

- First, the browser checks its own **cache** — *"Have I visited this before? Do I already know the IP?"*
- If not, it asks a **DNS resolver** (usually run by your internet provider, or a public one like Google's `8.8.8.8`)
- The resolver looks it up and replies: *"example.com is at 93.184.216.34"*

> **Think of it like this:** You want to call a restaurant. You don't remember the number, so you search "Pizza House" in your contacts — boom, number found. DNS does that for your browser.

DNS's job is done. It hands off an IP address and steps back. Everything from here is about reaching that address.

---

## Step 2 — Protocol: Picking How to Talk

Now the browser knows *where* to go. But before sending anything, it needs to decide *how* to communicate.

This is where **protocols** come in. A protocol is just a set of rules — an agreed way for two machines to talk to each other. Think of it like choosing between a phone call and a text message. Same person, different communication style.

The two most common transport protocols are **TCP** and **UDP**:

**TCP (Transmission Control Protocol)**
Reliable and ordered. Before any data is sent, both sides do a small "handshake" — they check that the connection is alive and ready. Every packet is tracked. If something gets lost, it's re-sent. TCP is like a phone call — you wait for the other person to pick up before you start talking.

**UDP (User Datagram Protocol)**
Fast but no guarantees. Data is fired off without waiting for confirmation. If a packet is lost, too bad — it's not re-sent. UDP is like shouting across a room. Quick, but not always perfect. Great for live video or gaming where speed > accuracy.

For web browsing, **TCP is used** — you want your webpage to arrive complete and in order.

---

## Step 3 — Opening a Connection (Ports)

TCP is chosen. Now the browser opens a **connection** to the server's IP address — but it needs one more thing: a **port**.

A port is like an apartment number. The IP gets you to the building (the server), but the port tells you which door to knock on.

Different services run on different ports by convention:

- `Port 80` → HTTP (regular, unencrypted web)
- `Port 443` → HTTPS (encrypted web)

So the browser is now knocking on `93.184.216.34:443` — saying *"I'd like to have a conversation, please."*

---

## Step 4 — Firewall: The Security Guard

The request doesn't reach the server directly. First, it meets the **firewall**.

A firewall is a gatekeeper. It looks at every incoming connection and decides: *allow or block?*

It checks three things:
- **Where is this coming from?** (source IP) — Is this a known bad actor?
- **Which port are they knocking on?** — Is that port supposed to be open?
- **What protocol is being used?** — TCP or UDP?

If everything looks fine, the firewall waves it through. If something seems off — blocked.

One important thing: the firewall **cannot read the content** of the request. It only sees the outer envelope — not the letter inside. That's by design. It works at a lower level, before any HTTP content is visible.

---

## Step 5 — TLS Handshake: Locking the Door

If you're using HTTPS (which most sites do today), the browser and server now do a **TLS handshake** before any real data is exchanged.

TLS stands for *Transport Layer Security*. Here's what happens:

1. Server sends its **SSL certificate** — proof of identity, like showing an ID card
2. Browser verifies it — *"Yes, this is really example.com"*
3. Both sides agree on an **encryption method**
4. An encrypted tunnel is established

Everything from this point is encrypted. Nobody in between can read the data — not your ISP, not a hacker on the same Wi-Fi.

> The lock icon in your browser's address bar means TLS is active.

---

## Step 6 — Reverse Proxy: The Smart Receptionist

The request — now encrypted and authenticated — finally arrives at the server. But it doesn't immediately reach the code that handles your request. It first meets the **reverse proxy**.

A reverse proxy is a middleman that sits in front of your backend servers. It receives all incoming requests and decides where to send each one.

It routes based on rules like:

- **Domain** → `api.example.com` goes to the API service, `example.com` goes to the main web app
- **Path** → `/images/` goes to the image server, `/checkout` goes to the payments service
- **Load balancing** → if there are 3 backend servers, spread the traffic evenly across them

> **Think of it like a hotel waiter.** You (the browser) sit at a table and place an order. The waiter (reverse proxy) takes the order and walks it to the right chef (backend server). The chef doesn't know your name, your table number, or anything about you — they just see the order and start cooking.

Common reverse proxies: **Nginx**, **Caddy**, **HAProxy**.

---

## Step 7 — Backend: Where the Work Happens

The reverse proxy forwards your request to the right **backend service**.

The backend could be running:
- On the same machine (accessed via `localhost`)
- On a different server (accessed via its internal IP)
- Inside a container or a Kubernetes pod

The backend processes the request:
1. Runs the business logic — *"what should this response be?"*
2. Talks to a **database** if it needs to fetch or store data
3. Maybe checks a **cache** for faster lookups
4. Builds the response

---

## Step 8 — The Response Travels Back

The backend hands the response to the reverse proxy. The reverse proxy sends it back through the same TCP connection. The browser receives it and renders the page.

```
You click → DNS → TCP + Port → Firewall → TLS → Reverse Proxy → Backend
                                                                     ↓
You see the page ← Browser renders ← Response travels back the same way
```

The whole thing — start to finish — happens in under a second.

---

*Video-1 · System Design Notes*
