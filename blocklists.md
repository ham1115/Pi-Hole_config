# 📋 Blocklists

Pi-hole's effectiveness depends largely on its blocklists. This document outlines the blocklists used in this setup and the rationale for each.

---

## Default Blocklist

Pi-hole ships with a default blocklist (StevenBlack's Unified Hosts) which covers the most common ad and tracking domains. This is enabled out of the box.

---

## Additional Blocklists Used

| List | URL | Purpose |
|---|---|---|
| Steven Black Unified | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | Ads + malware (base list) |
| OISD Full | `https://big.oisd.nl` | Comprehensive ads, tracking, phishing |
| Hagezi Pro | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt` | Aggressive ad/tracker blocking |
| No-Track | `https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-blocklist.txt` | Tracker-focused |

---

## How to Add a Blocklist in Pi-hole

1. Go to the Pi-hole admin dashboard (`http://<your-ip>/admin`)
2. Navigate to **Group Management → Adlists**
3. Paste the blocklist URL into the **Address** field
4. Click **Add**
5. Go to **Tools → Update Gravity** to download and apply the list

---

## Blocklist Selection Rationale

- **Breadth over depth for ads**: The OISD and StevenBlack lists cover the vast majority of known ad networks with minimal false positives.
- **Avoid overly aggressive lists**: Some lists block legitimate services (e.g., telemetry from tools you intentionally use). Tested each list incrementally to check for breakage.
- **Tracker blocking**: Added a dedicated tracker list since many trackers are hosted on domains that aren't strictly "ads" and wouldn't be caught by ad-only lists.

---

## Managing False Positives (Whitelisting)

If a legitimate site or service breaks after enabling a blocklist, you can whitelist the domain:

1. Check the Pi-hole **Query Log** to find which domain is being blocked
2. Go to **Domains → Add Domain → Whitelist**
3. Add the domain and submit

Common domains that sometimes require whitelisting:
- CDN or API endpoints for legitimate services
- Some update/telemetry endpoints for software you intentionally use

---

## Notes

- Blocklists are refreshed automatically when you run **Update Gravity**
- It's good practice to update gravity periodically (weekly or monthly) as new ad domains are added to lists
- Too many lists can slow down gravity updates and bloat the database without meaningful additional coverage — quality over quantity
