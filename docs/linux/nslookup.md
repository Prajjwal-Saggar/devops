# DNS Cheat Sheet & DevOps Incident Workflow

---

## 1. Core Concepts & Definitions

### DNS (Domain Name System)

**DNS (Domain Name System)** is the internet's phonebook. It translates human-friendly domain names such as:

```text
learnlinux.tv
```

into numerical IP addresses such as:

```text
172.105.29.178
```

This allows users to access websites using memorable domain names instead of IP addresses.

---

### Domain vs. Subdomain

| Concept       | Description                                        | Example           |
| ------------- | -------------------------------------------------- | ----------------- |
| **Domain**    | The main website identity or primary address       | `google.com`      |
| **Subdomain** | A specific section, service, or branch of a domain | `mail.google.com` |

For example:

```text
mail.google.com
│    │      │
│    │      └── Top-level domain (TLD)
│    └───────── Domain
└────────────── Subdomain
```

---

### TTL (Time To Live)

**TTL (Time To Live)** specifies how long a DNS record can be cached before the DNS resolver should request a fresh copy.

TTL is measured in **seconds**.

For example:

```text
TTL = 300
```

means the record can be cached for:

```text
300 seconds = 5 minutes
```

A lower TTL allows DNS changes to propagate more quickly, while a higher TTL reduces the number of DNS queries.

---

### Port 53

DNS traditionally uses:

```text
Port: 53
```

DNS queries primarily use **UDP** because it is fast and has low overhead.

DNS can also use **TCP port 53**, particularly for situations such as large responses or zone transfers.

---

### Authoritative vs. Non-Authoritative DNS

#### Authoritative

An **authoritative DNS server** is a server that holds the official DNS records for a domain.

Example:

```text
example.com → 93.184.216.34
```

The authoritative server is responsible for providing the official answer for that domain.

#### Non-Authoritative

A **non-authoritative answer** comes from a DNS resolver that obtained the information from another DNS server and may have it cached.

For example:

```text
Your Computer
      ↓
Local DNS Resolver
      ↓
Authoritative DNS Server
```

The local resolver may return a cached answer without contacting the authoritative server every time.

---

# 2. Essential CLI Commands for DevOps

## `nslookup`

`nslookup` is a simple, cross-platform command-line tool used to perform DNS lookups.

### Basic Usage

```bash
nslookup example.com
```

It can be used to quickly determine the IP address associated with a domain.

Example:

```text
$ nslookup example.com

Name:    example.com
Address: 93.184.216.34
```

---

## `dig`

`dig` is a more detailed DNS diagnostic tool commonly used on Linux and Unix-like systems.

### Basic Usage

```bash
dig example.com
```

It provides information such as:

* DNS response status
* Answer records
* TTL values
* Query execution time
* DNS server used
* Record types

Example structure:

```text
;; ->>HEADER<<- opcode: QUERY, status: NOERROR

;; QUESTION SECTION:
;example.com.       IN      A

;; ANSWER SECTION:
example.com.    300     IN      A       93.184.216.34
```

`NOERROR` indicates that the DNS query was successfully processed.

---

## Query a Specific DNS Server

You can explicitly specify which DNS resolver should answer your query.

For example, to query Google's public DNS:

```bash
dig @8.8.8.8 example.com
```

The syntax is:

```bash
dig @DNS_SERVER DOMAIN
```

For example:

```bash
dig @1.1.1.1 example.com
```

queries Cloudflare's DNS resolver.

This is useful when troubleshooting DNS because it allows you to compare responses from different resolvers.

---

## Common Public DNS Servers

| Provider              | IP Address            | Primary Feature            |
| --------------------- | --------------------- | -------------------------- |
| **Google Public DNS** | `8.8.8.8` / `8.8.4.4` | Reliable global resolution |
| **Cloudflare DNS**    | `1.1.1.1` / `1.0.0.1` | Speed and privacy          |
| **Quad9 DNS**         | `9.9.9.9`             | Threat blocking            |

---

# 3. DevOps Real-World Incident Workflow

## Scenario

A website or microservice becomes unreachable after a:

* Cloud server migration
* Blue/Green deployment
* Infrastructure change
* DNS configuration change

The first thing to investigate is whether DNS is directing traffic to the correct server.

---

## Step 1: Identify the Traffic Destination

Run:

```bash
dig shop.company.com
```

Look at the **ANSWER SECTION**.

For example:

```text
;; ANSWER SECTION:
shop.company.com.    300    IN    A    203.0.113.20
```

The returned IP address is:

```text
203.0.113.20
```

Compare this IP with the IP address of the new server in your cloud provider or infrastructure platform.

For example:

```text
DNS IP:          203.0.113.20
New Server IP:   203.0.113.20
```

If they match, DNS is pointing to the expected server.

---

## Step 2: Diagnose Caching vs. Misconfiguration

Query Google's public DNS directly:

```bash
dig @8.8.8.8 shop.company.com
```

Then compare it with your normal DNS lookup:

```bash
dig shop.company.com
```

### Case 1: Local DNS Shows OLD IP, Google Shows NEW IP

Example:

```text
Local DNS:
shop.company.com → 203.0.113.10

Google DNS:
shop.company.com → 203.0.113.20
```

This suggests that the local resolver may still have the **old DNS record cached**.

Check the TTL:

```bash
dig shop.company.com
```

Look at the record:

```text
shop.company.com.    120    IN    A    203.0.113.10
```

The TTL indicates how long the cached record may remain valid.

The old record should eventually expire, after which the resolver can retrieve the updated record.

---

### Case 2: Both Show the OLD IP

If both:

```bash
dig shop.company.com
```

and:

```bash
dig @8.8.8.8 shop.company.com
```

return the old IP, the problem is probably **not just local caching**.

Possible causes include:

* DNS record was not updated
* DNS provider configuration is incorrect
* Wrong DNS zone was modified
* Nameservers are pointing to a different DNS provider
* The new record was created incorrectly

At this point, inspect the DNS configuration at the authoritative DNS provider.

---

# 4. DNS Migration Best Practice

Before performing a planned server migration or traffic cutover, reduce the DNS TTL.

For example:

```text
Before migration:

TTL = 86400 seconds
       ↓
24 hours
```

Reduce it ahead of time:

```text
TTL = 60 seconds
```

Then perform the migration or DNS cutover.

After the migration is stable, the TTL can be increased again.

### Typical Migration Flow

```text
24+ hours before migration
        ↓
Lower DNS TTL
        ↓
Wait for old TTL to expire
        ↓
Migrate server
        ↓
Update DNS record
        ↓
Monitor traffic
        ↓
Verify DNS resolution
        ↓
Increase TTL again
```

The important point is that **lowering the TTL immediately before changing the record does not instantly invalidate caches that were created under the previous TTL**. The lower TTL should be configured sufficiently in advance.

---

# 5. Quick DNS Troubleshooting Cheat Sheet

| Goal                 | Command                    |
| -------------------- | -------------------------- |
| Basic DNS lookup     | `nslookup example.com`     |
| Detailed DNS lookup  | `dig example.com`          |
| Query Google DNS     | `dig @8.8.8.8 example.com` |
| Query Cloudflare DNS | `dig @1.1.1.1 example.com` |
| Query Quad9 DNS      | `dig @9.9.9.9 example.com` |
| Check A record       | `dig example.com A`        |
| Check MX records     | `dig example.com MX`       |
| Check NS records     | `dig example.com NS`       |
| Check TXT records    | `dig example.com TXT`      |

---

# 6. Incident Decision Tree

```text
Website unreachable
        │
        ▼
Run: dig shop.company.com
        │
        ▼
Is the returned IP correct?
       / \
     NO   YES
     │     │
     │     ▼
     │   Check server,
     │   firewall, ports,
     │   load balancer, etc.
     │
     ▼
Run:
dig @8.8.8.8 shop.company.com
        │
        ▼
Does Google DNS show the new IP?
       / \
     YES  NO
      │    │
      │    ▼
      │  Check DNS provider,
      │  authoritative records,
      │  nameservers, etc.
      │
      ▼
Likely caching/resolver
propagation issue
```

---

# 7. Key Takeaways

1. **DNS translates domain names into IP addresses.**
2. **TTL controls how long DNS responses can be cached.**
3. **Port 53 is used for DNS, primarily over UDP, with TCP also used in certain cases.**
4. **`nslookup` is useful for quick DNS checks.**
5. **`dig` provides more detailed DNS troubleshooting information.**
6. **`dig @8.8.8.8 domain.com` lets you query Google's public DNS directly.**
7. **If different DNS resolvers return different IPs, caching or DNS propagation may be involved.**
8. **If multiple independent resolvers return the old IP, investigate the authoritative DNS configuration.**
9. **Lower DNS TTL well before a planned migration to make a future DNS cutover faster.**
10. **DNS is only one part of troubleshooting an unreachable service; after confirming DNS, investigate networking, load balancers, firewalls, ports, and the application itself.**
