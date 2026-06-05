# Pi-hole Network-Wide Ad Blocker — Homelab Project
A self-hosted DNS sinkhole deployed via Docker on a repurposed Lenovo IdeaPad running Ubuntu Linux, configured as the primary DNS server for the entire home network.

## Project Overview

This project transforms an old laptop into a dedicated network security appliance that blocks ads, trackers, and malicious domains at the DNS level, before they ever reach any device on the network.

Unlike browser-based ad blockers that only protect one device, this setup intercepts DNS queries from every device on the network (phones, smart TVs, IoT devices, PCs) with zero per-device configuration required.

## Tech Stack
| Component  | Details|
| ------------- | ------------- |
| Host OS  | Ubuntu Linux  |
| Containerization  | Docker |
| DNS Sinkhole  | Pi-Hole (latest)  |
| DNS Protocol  | Standard DNS (port 53) |
| Network scope | Entire LAN via router DNS config |

## Setup Summary
1. Installed Ubuntu on a repurposed Lenovo IdeaPad
2. Installed Docker Engine on Ubuntu
3. Deployed Pi-hole as a Docker container with a persistent volume for config/data
4. Assigned a static IP to the laptop on the LAN
5. Configured the home router to use the laptop's IP as the Primary DNS Server
6. Verified network-wide DNS filtering across all connected devices

Full step-by-step setup guide in docs/setup.md

## Results
- ✅ Ads and trackers blocked network-wide across all devices
- ✅ Smart TV ad domains blocked (no per-TV configuration needed)
- ✅ Mobile devices protected without installing any app
- ✅ Pi-hole dashboard provides real-time DNS query analytics
- ✅ Custom blocklists added for malicious and phishing domains

## Security Notes
- All real IP addresses, MAC addresses, and network-specific identifiers have been anonymized in this documentation
- No credentials or API keys are stored in this repository
- Pi-hole admin panel is accessible only within the LAN (not exposed externally)

## Further reading
[Pi-hole Official Docs](https://docs.pi-hole.net/)
[Pi-hole Docker Hub](https://hub.docker.com/r/pihole/pihole)
[Ubuntu Server Guide](https://ubuntu.com/server/docs)
