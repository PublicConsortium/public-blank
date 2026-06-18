# Public Blank

A free, public static parking page service. If you own a domain that needs a default landing page — registered but not yet built out, or pointed here while you are between hosts — `public-blank.com` serves a clean HTTPS placeholder for it. No ads, no trackers, no third-party content.

| | |
|---|---|
| **Transport** | HTTPS (HTTP/2 and HTTP/3) |
| **Network** | IPv4 and IPv6 |
| **Logging** | None |
| **TLS** | Per-domain Let's Encrypt (ECC P-256) |
| **Cost** | $0 |

**Website:** [public-blank.com](https://public-blank.com/)

---

## Table of contents

- [Quick Start](#quick-start)
- [DNS Setup](#dns-setup)
- [Privacy](#privacy)
- [How to Use](#how-to-use)
- [TLS](#tls)
- [Features](#features)
- [Infrastructure](#infrastructure)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Acceptable Use](#acceptable-use)
- [Other Projects](#other-projects)
- [Contact and Support](#contact-and-support)

---

## Quick Start

1. Email [root@public-common.com](mailto:root@public-common.com) with the domain you want parked.
2. An nginx `server` block is added for your domain, and a TLS certificate is issued via Let's Encrypt (DNS-01).
3. You receive the IP addresses and DNS record instructions.
4. Set the `A` and `AAAA` records at your DNS provider.
5. Once DNS propagates, your domain serves the parking page over HTTPS.

---

## DNS Setup

The host is single-node, dual-stack:

| | |
|---|---|
| **IPv4** | `37.27.97.202` |
| **IPv6** | `2a01:4f9:3070:145b::1` |

For your domain (e.g. `example.com`), the apex needs:

```
example.com.        IN  A     37.27.97.202
example.com.        IN  AAAA  2a01:4f9:3070:145b::1
```

If you want `www` to park here as well:

```
www.example.com.    IN  A     37.27.97.202
www.example.com.    IN  AAAA  2a01:4f9:3070:145b::1
```

If your registrar's UI uses record forms rather than zone-file syntax, the equivalent is: type `A`, host `@`, value `37.27.97.202`; then type `AAAA`, host `@`, value `2a01:4f9:3070:145b::1`.

Confirm propagation against any validating resolver — for example [public-rdns.com](https://public-rdns.com/):

```bash
dig @public-rdns.com +short example.com A
dig @public-rdns.com +short example.com AAAA
```

If you host authoritative DNS on [public-adns.com](https://public-adns.com/), add the two records to your zone file and bump the serial.

---

## Privacy

A parking page should do nothing interesting. We do not log individual page views. No visitor data is stored, sold, or shared with third parties.

- **No access logging** — page views are not written to disk.
- **No analytics, no cookies, no third-party content** — the parking page is static HTML.
- **No ads, no monetisation** — this exists so you do not have to host a parking page yourself.
- **Encrypted storage** — data at rest is protected with ZFS native encryption.
- **No shell history** — operator sessions leave no command history on the server.
- **Headless server** — no physical console exposed to the public internet.

---

## How to Use

There is no self-service portal. To park a domain, email [root@public-common.com](mailto:root@public-common.com) with the FQDN. The operator adds an nginx `server` block, issues a certificate, and replies with the DNS values under [DNS Setup](#dns-setup).

To unpark a domain, change its DNS to point elsewhere. The parking record on this server can be removed on request.

Subdomains use the same flow — set `A`/`AAAA` on the subdomain and email the FQDN.

---

## TLS

Each parked domain gets its own Let's Encrypt certificate, issued via ACME with the DNS-01 challenge and an ECC P-256 key. Certificates are managed centrally on the [Public Consortium](https://public-consortium.com/) management host and synchronised to this host before nginx reloads.

- HTTPS is on by default, on TCP 443 (TLS).
- Plain HTTP on port 80 is also served. Use the `https://` URL when you want TLS — there is no automatic redirect.
- HTTP/2 and HTTP/3 (QUIC) are enabled where the client supports them.
- Certificates auto-renew. There is nothing for you to do.

---

## Features

- HTTPS with HTTP/2 and HTTP/3
- IPv4 and IPv6
- Per-domain Let's Encrypt certificates (ECC P-256), auto-renewed
- Hardened response headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`
- HTTP `GET` only — every other method is denied at the web tier
- Static HTML, no scripts, no third-party assets
- No charge, no signup, no API key

---

## Infrastructure

- **OS:** FreeBSD
- **Web server:** nginx with HTTP/2 and HTTP/3
- **Filesystem:** ZFS with native encryption
- **Certificates:** issued centrally on `root.public-common.com` via `acme.sh`, synced to this host with `rsync`
- **nginx layout:** one `server` block per parked domain, each pinned to its own certificate

---

## Troubleshooting

### The browser shows a certificate warning

Most often the DNS is pointed here but the certificate has not been issued for that name yet. Issuance happens on the next ACME loop after the request — wait a few minutes and reload, or email to confirm.

### The page loads from one network but not another

Usually your AAAA record is set but the client is reaching us over a broken IPv6 path (or vice versa). Check both `dig A` and `dig AAAA` for your domain, and confirm both addresses match [DNS Setup](#dns-setup).

### I see a different site at the IP

The host serves many domains from the same IP — nginx routes by SNI / Host header. Reaching the IP directly without an SNI returns a default response, not your domain. That is normal.

### My old hosting still shows up

DNS caches. Wait for your previous record's TTL to expire, or flush the resolver you are testing from.

---

## FAQ

**Is this really free?**  
Yes. There is no charge, no signup, no API key. Donations via Bitcoin are appreciated but not required — see [Contact and Support](#contact-and-support).

**Can I customise the parking page?**  
Not currently. The page is a single shared template — that is what keeps the service simple to operate. If you need a customised landing page, host your own static site and only point at `public-blank.com` while you are between providers.

**Can you host my real site here?**  
No. This is parking only — static HTML, one shared placeholder.

**Will my domain show ads or "for sale" banners?**  
No. There is no monetisation on this service.

**Do you log visitors?**  
No. Access requests are not written to disk.

**How do I unpark?**  
Change your DNS to point at your new host. Optionally, email and the `server` block for your domain will be removed.

**What's the SLA?**  
Best-effort. The service is operated as a public good. A parking page being briefly unreachable is a smaller problem than a production site being down, so this is the right tier of service for the use case.

---

## Acceptable Use

- Park your own domains, not domains you do not control.
- Do not use the parking page as part of a phishing or malware infrastructure. Such domains are removed without notice and reported.
- Do not try to wrap arbitrary content through the parking host — the page is static and there is nothing to wrap.
- Abusive sources may be rate limited or blocked without notice.

---

## Other Projects

| Site | Service |
|------|---------|
| [public-consortium.com](https://public-consortium.com/) | Project home and operations |
| [public-adns.com](https://public-adns.com/) | Public authoritative DNS service |
| [public-rdns.com](https://public-rdns.com/) | Public recursive DNS service |
| [public-blank.com](https://public-blank.com/) | Public static / parking service (this site) |
| [public-repo.com](https://public-repo.com/) | Public mirror service |
| [public-utc.com](https://public-utc.com/) | Public NTP / NTS time service |

---

## Contact and Support

For parking requests, abuse reports, or operational issues, email [root@public-common.com](mailto:root@public-common.com).

Discord: [Public Consortium](https://discord.gg/Hjgz5EWb)

Support via BTC: `bc1qxfcfkvgyjs6pq6cdmv6syx5s6pc9dn3wsuewjq`
