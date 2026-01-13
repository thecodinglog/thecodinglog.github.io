---
layout: post
title: DNS Records - Your Memory Refresher Guide 🌐
description: DNS Records - Your Memory Refresher Guide 🌐
date:   2026-01-13 10:50:00 +0900
author: Jeongjin Kim
categories: Networking
tags:	Networking
---

Ever stared at your DNS settings panel and thought "Wait, what's the difference between an A record and a CNAME again?" You're definitely not alone. After a few weeks away from DNS configuration, all those record types start blending together. This guide will help you rebuild that mental model and understand what's actually happening when you set up your domain.

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

# What's DNS Anyway?

Think of DNS (Domain Name System) as the internet's phonebook. When you type `example.com` into your browser, DNS translates that human-friendly name into an IP address like `192.0.2.1` that computers actually understand.

DNS records are the individual entries in this phonebook—each one contains specific information about how to handle requests for your domain.

# The Big Picture: Record Types at a Glance

Before diving deep, here's a quick reference table you'll want to bookmark:

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps domain to IPv4 address | example.com → 123.123.123.123 |
| **AAAA** | Maps domain to IPv6 address | example.com → 2001:0db8:85a3::7334 |
| **CNAME** | Creates domain alias | www.example.com → example.com |
| **NS** | Specifies authoritative nameservers | example.com → ns1.dnsprovider.com |

Now let's break these down properly.

# A Record: The Foundation

The A record (Address Record) is the most fundamental DNS record. It's what actually connects your domain name to a server's location.

**What it does:** Maps a domain name directly to an IPv4 address

**Why you need it:** This is how browsers find your website

**Example:**
```
example.com    A    192.0.2.1
```

When someone types `example.com` into their browser, DNS looks up the A record and says "Ah, that's at 192.0.2.1!" Then the browser connects to that IP address.

### Real-World Scenario

Imagine you're launching a new web application. You've got a server at `203.0.113.10` and you just registered `myapp.com`. You create an A record:

```
myapp.com    A    203.0.113.10
```

Now when users visit `myapp.com`, they're connecting directly to your server. Simple, clean, effective.

# AAAA Record: The IPv6 Cousin

The AAAA record (pronounced "quad-A") is basically the A record's modern sibling.

**What it does:** Maps a domain name to an IPv6 address

**Why it exists:** IPv4 addresses are running out. IPv6 solves this with a massive address space.

**Example:**
```
example.com    AAAA    2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

### A vs AAAA: The Quick Comparison

| Feature | A Record | AAAA Record |
|---------|----------|-------------|
| IP Version | IPv4 (32-bit) | IPv6 (128-bit) |
| Address Format | 203.0.113.10 | 2001:db8::1 |
| Address Space | ~4.3 billion addresses | 340 undecillion addresses (yeah, that's a real number) |
| Current Usage | Universal | Growing rapidly |

### Why You Should Care About IPv6

Here's the thing: IPv4 addresses ran out years ago. We're currently using various workarounds (like NAT) to keep things running, but IPv6 is the future. Major platforms—Google, Facebook, Netflix—are already fully IPv6 enabled.

**Pro tip:** Run both A and AAAA records. Some networks and devices prefer IPv6, while others still rely on IPv4. Having both ensures maximum compatibility.

```
example.com    A       203.0.113.10
example.com    AAAA    2001:db8:85a3::1
```

# CNAME Record: The Alias Master

CNAME (Canonical Name) records create aliases. Instead of pointing to an IP address, they point to another domain name.

**What it does:** Creates an alias from one domain to another

**Why you need it:** Avoid duplicate configuration and simplify management

**Example:**
```
www.example.com    CNAME    example.com
```

When someone visits `www.example.com`, DNS says "That's actually just `example.com`" and then looks up the A record for `example.com`.

### When CNAME Shines

**Scenario 1: Subdomain Management**

You're running multiple services under one domain:

```
www.example.com      CNAME    example.com
blog.example.com     CNAME    example.com
shop.example.com     CNAME    example.com
```

Now if you change servers (and thus your IP address), you only need to update the A record for `example.com`. All the CNAMEs automatically follow.

**Scenario 2: CDN Integration**

You want to use Cloudflare's CDN:

```
cdn.example.com    CNAME    example.cloudflarecdn.com
```

Your CDN provider can manage the underlying infrastructure while you keep a simple CNAME in your DNS.

### The CNAME Gotcha

Here's something that trips people up constantly: **you cannot use a CNAME alongside other records for the same hostname**.

This is **invalid**:
```
example.com    A         192.0.2.1
example.com    CNAME     somewhere.else.com    ❌ CONFLICT!
```

But this is **fine**:
```
example.com        A         192.0.2.1
www.example.com    CNAME     example.com          ✅ Different hostnames
```

Why? The DNS specification requires that a CNAME record be the only record for that exact name. It's an exclusive relationship.

# NS Record: The Authority Delegation

NS records (Name Server records) are meta-records—they control which DNS servers are authoritative for your domain.

**What it does:** Specifies which nameservers handle DNS queries for your domain

**Why it matters:** These records determine where all DNS lookups for your domain are sent

**Example:**
```
example.com    NS    ns1.dnsprovider.com
example.com    NS    ns2.dnsprovider.com
```

### How NS Records Work

When someone queries your domain:

1. Their DNS resolver asks the root DNS servers "Who handles `.com`?"
2. Root servers respond with the `.com` TLD nameservers
3. The resolver asks TLD servers "Who handles `example.com`?"
4. TLD servers respond with your NS records: `ns1.dnsprovider.com` and `ns2.dnsprovider.com`
5. The resolver finally asks those nameservers for the actual A, AAAA, CNAME records

**Key insight:** NS records are like forwarding addresses. They tell the world "For anything about this domain, ask these servers."

### Multiple Nameservers: Redundancy

You'll typically see 2-4 NS records for reliability:

```
example.com    NS    ns1.dnsprovider.com
example.com    NS    ns2.dnsprovider.com
example.com    NS    ns3.dnsprovider.com
example.com    NS    ns4.dnsprovider.com
```

If one nameserver goes down, the others can still respond. This is critical for availability.

# Understanding the Host Field: @, www, and *

When you're configuring DNS, the "host" or "name" field can be confusing. Let's demystify the common symbols:

### The @ Symbol (Root Domain)

`@` represents your root domain—the domain itself without any subdomain prefix.

```
@    A    192.0.2.1
```

This creates a record for `example.com` (not `subdomain.example.com`).

### The www Prefix

This is the classic subdomain for web services:

```
www    CNAME    example.com
```

Creates a record for `www.example.com` pointing to `example.com`.

### The * Wildcard

The wildcard matches any subdomain that doesn't have a specific record:

```
*    A    192.0.2.1
```

This means `anything.example.com` will resolve to `192.0.2.1` unless there's a more specific record.

**Example with specifics:**

```
example.com          A         192.0.2.1
www.example.com      CNAME     example.com
api.example.com      A         192.0.2.2
*                    A         192.0.2.1
```

Results:
- `example.com` → 192.0.2.1 (root A record)
- `www.example.com` → 192.0.2.1 (CNAME to root)
- `api.example.com` → 192.0.2.2 (specific A record)
- `random.example.com` → 192.0.2.1 (wildcard catches it)
- `foo.bar.example.com` → 192.0.2.1 (wildcard catches it)

### Common Host Field Patterns

| Host | Creates | Common Use |
|------|---------|------------|
| `@` | example.com | Root domain |
| `www` | www.example.com | Web services |
| `mail` | mail.example.com | Mail server |
| `api` | api.example.com | API endpoint |
| `ftp` | ftp.example.com | File transfer |
| `*` | *.example.com | Wildcard catchall |


<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>

# Industry Trends: What's Happening Now

### Cloud-Native DNS Services

Traditional DNS hosting is giving way to cloud-native solutions. AWS Route 53, Cloudflare DNS, and Google Cloud DNS offer:

- **Global distribution:** Faster lookups from anywhere
- **Advanced traffic management:** Geolocation routing, weighted routing
- **Integrated security:** Built-in DDoS protection, DNSSEC
- **Programmable infrastructure:** Manage DNS via API

If you're running anything beyond a hobby project, using a specialized DNS service is practically mandatory now.

### CDN Integration is Standard

Every major website uses CDN services like Cloudflare, Akamai, or Fastly. CNAME records make this integration trivial:

```
cdn.example.com    CNAME    example.cdn.cloudflare.net
```

Your CDN provider handles the global edge network while you maintain a simple DNS record.

### IPv6 Adoption is Accelerating

According to Google's IPv6 statistics, over 40% of users now access their services via IPv6. For mobile networks, it's even higher—often 70-80%.

**Reality check:** If you're launching a mobile app or targeting markets with high mobile penetration (like India, China, or most of Asia), IPv6 isn't optional anymore.

```
example.com    A       203.0.113.10      # IPv4
example.com    AAAA    2001:db8::1       # IPv6
```

Run both. Always.

### DNS Security (DNSSEC) is Going Mainstream

DNSSEC adds cryptographic signatures to DNS records, preventing spoofing attacks. Major TLDs (`.com`, `.net`, `.org`) support it, and hosting providers are making it easier to enable.

Without DNSSEC, an attacker could redirect `yourbank.com` to their fake site. With DNSSEC, such tampering is cryptographically provable and rejected.

# Practical Tips: Getting It Right

### For New Projects (Short-Term Strategy)

1. **Start with the basics:** Set up A records and one CNAME for www
   ```
   example.com        A         192.0.2.1
   www.example.com    CNAME     example.com
   ```

2. **Add IPv6 early:** Don't wait until you "need" it
   ```
   example.com    AAAA    2001:db8::1
   ```

3. **Use a proper DNS provider:** Don't rely on your domain registrar's free DNS—it's often slow and limited

4. **Set reasonable TTLs:** During initial setup, use short TTLs (300 seconds) so mistakes can be corrected quickly

### For Growing Services (Long-Term Strategy)

1. **Choose enterprise DNS:** Evaluate AWS Route 53, Cloudflare, or Google Cloud DNS based on:
   - Global presence
   - Security features (DDoS protection, DNSSEC)
   - Traffic routing capabilities
   - API integration for automation

2. **Implement monitoring:** DNS downtime is total downtime. Monitor your NS records and query response times

3. **Plan for IPv6 transition:** Government agencies, large enterprises, and global services should aggressively adopt IPv6

4. **Enable DNSSEC:** Especially for financial services, healthcare, or any site handling sensitive data

5. **Document your DNS architecture:** Future you (or your successor) will thank you

### A Real-World Example: E-Commerce Site

Let's say you're building `shop.example.com` with these requirements:
- Main site on AWS
- CDN for static assets
- Separate API server
- Email via Google Workspace
- IPv4 and IPv6 support

Your DNS configuration might look like:

```
; Root domain
example.com             A       203.0.113.10
example.com             AAAA    2001:db8::1

; Web services
www.example.com         CNAME   example.com
shop.example.com        A       203.0.113.20
shop.example.com        AAAA    2001:db8::2

; CDN for assets
cdn.example.com         CNAME   shop.cloudfront.net

; API server
api.example.com         A       203.0.113.30
api.example.com         AAAA    2001:db8::3

; Email (MX records - not covered here, but you get the idea)
example.com             MX      10 aspmx.l.google.com

; Nameservers
example.com             NS      ns1.dnsprovider.com
example.com             NS      ns2.dnsprovider.com
example.com             NS      ns3.dnsprovider.com
```

# Common Mistakes to Avoid

### Mistake 1: Using CNAME at the Root

This is **wrong**:
```
example.com    CNAME    somewhere.else.com    ❌
```

The DNS specification doesn't allow CNAME at the apex (root) domain because it conflicts with required records like NS and SOA.

**Solution:** Use an A record at the root, or use your DNS provider's ALIAS/ANAME feature if available.

### Mistake 2: Forgetting About TTL

TTL (Time To Live) controls how long DNS records are cached. Setting it too high means changes take forever to propagate. Too low means unnecessary load on your DNS servers.

**Good practice:**
- During active development: 300 seconds (5 minutes)
- For stable production: 3600 seconds (1 hour)
- Before making changes: Lower TTL 24 hours in advance

### Mistake 3: Not Having Redundant Nameservers

Relying on a single NS record is asking for trouble. Always have at least two, preferably on different networks:

```
example.com    NS    ns1.dnsprovider.com
example.com    NS    ns2.dnsprovider.com
```

### Mistake 4: Ignoring IPv6

"Nobody uses IPv6 yet" is simply false. A significant portion of mobile users are IPv6-only or IPv6-preferred. If you're not providing AAAA records, you're degrading their experience or blocking them entirely.

# Testing Your DNS Configuration

Before you call it done, test your configuration:

### Using dig (Command Line)

```bash
# Check A record
dig example.com A

# Check AAAA record
dig example.com AAAA

# Check NS records
dig example.com NS

# Check CNAME
dig www.example.com CNAME

# Full trace (see the entire resolution path)
dig +trace example.com
```

### Using nslookup (Windows)

```cmd
nslookup example.com
```

### Online Tools

- **DNS Checker:** dnschecker.org - Check propagation globally
- **What's My DNS:** whatsmydns.net - See DNS responses from different locations
- **DNS Lookup:** mxtoolbox.com - Comprehensive DNS testing

# Wrapping Up

DNS essentials in a nutshell:

1. **A records:** Domain → IPv4 address (the foundation)
2. **AAAA records:** Domain → IPv6 address (the future)
3. **CNAME records:** Domain alias (the convenience)
4. **NS records:** Nameserver delegation (the meta-layer)

Remember:
- Always use multiple NS records for redundancy
- Deploy both A and AAAA records for compatibility
- CNAME can't coexist with other records for the same host
- Test everything before going live
- Use enterprise DNS for production workloads

DNS might seem like plumbing—invisible infrastructure nobody thinks about—but when it breaks, everything stops. Take the time to understand it properly, and you'll save yourself (and your users) from a world of pain.

Bookmark this page for the next time you're staring at that DNS configuration panel wondering what all those acronyms mean. You'll thank yourself later! 🚀

**Subscribe [via RSS](/feed.xml)**


<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 컨텐츠내 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3234744071843247"
     data-ad-slot="1671969273"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>