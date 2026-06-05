# Architecture — Pi-hole Homelab

## Overview

This setup uses a repurposed Lenovo IdeaPad as a dedicated DNS server for the home network. Pi-hole runs inside a Docker container and intercepts all DNS queries from every connected device.

---

## Network Diagram

```
┌─────────────────────────────────────────────────────┐
│                   HOME NETWORK (LAN)                │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌────────────────┐   │
│  │  Laptop  │   │  Phone   │   │   Smart TV     │   │
│  └────┬─────┘   └────┬─────┘   └───────┬────────┘   │
│       │              │                 │            │
│       └──────────────┴─────────────────┘            │
│                       │                             │
│              DNS queries (port 53)                  │
│                       │                             │
│          ┌────────────▼─────────────┐               │
│          │   Lenovo IdeaPad         │               │
│          │   Ubuntu Linux           │               │
│          │   Static IP: [LAN IP]    │               │
│          │                          │               │
│          │  ┌─────────────────────┐ │               │
│          │  │  Docker Container   │ │               │
│          │  │  Pi-hole            │ │               │
│          │  │  - DNS Sinkhole     │ │               │
│          │  │  - Admin Dashboard  │ │               │
│          │  │  - Blocklists       │ │               │
│          │  └─────────────────────┘ │               │
│          └────────────┬─────────────┘               │
│                       │                             │
└───────────────────────┼─────────────────────────────┘
                        │
              Allowed queries forwarded
                        │
            ┌───────────▼──────────────┐
            │   Upstream DNS Resolver  │
            │   (e.g., 1.1.1.1 /       │
            │    8.8.8.8)              │
            └──────────────────────────┘
```

---

## How It Works

### 1. DNS Query Interception
Every device on the network is pointed to the IdeaPad's static LAN IP as its DNS server (configured at the router level). When any device wants to resolve a domain name, the query goes to Pi-hole first.

### 2. Blocklist Matching
Pi-hole checks the queried domain against its aggregated blocklists containing millions of known ad, tracker, and malicious domains.

- **Blocked domain** → Pi-hole returns `0.0.0.0` (a null/sinkhole response). The device gets no IP address, the connection never happens, and the ad/tracker never loads.
- **Allowed domain** → Pi-hole forwards the query to the configured upstream DNS resolver (e.g., Cloudflare `1.1.1.1`) and returns the real IP.

### 3. Network-Wide Coverage
Because the DNS configuration is set at the **router level**, every device that joins the network automatically uses Pi-hole, no client-side software or configuration needed. This includes:
- Smartphones and tablets
- Smart TVs and streaming sticks
- IoT devices (smart bulbs, cameras, etc.)
- Guest devices

---

## Design Decisions

| Decision | Rationale |
|---|---|
| Docker over bare-metal install | Easier to update, isolate, and redeploy without touching the host OS |
| Static IP via router DHCP reservation | Ensures the DNS server address never changes; simpler than static IP in OS network config |
| Primary DNS only (no secondary) | Secondary DNS would bypass Pi-hole on failure; intentionally left blank to ensure filtering is always enforced |
| Upstream: 1.1.1.1 | Privacy-focused, fast, and reliable upstream resolver |

---

## Limitations & Considerations

- **Single point of failure**: If the IdeaPad goes down, DNS resolution fails network-wide. Mitigation: keep the device plugged in and configure auto-restart for the Docker container (`restart: unless-stopped`).
- **HTTPS/DNS-over-HTTPS (DoH)**: Some browsers and apps use hardcoded DoH endpoints that bypass traditional DNS. Pi-hole cannot block these without additional firewall rules.
- **Encrypted DNS**: Devices using DNS-over-TLS (DoT) or DoH natively can circumvent the sinkhole.
