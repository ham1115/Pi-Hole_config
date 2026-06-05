# 📝 Lessons Learned

A candid account of the challenges encountered during this project and how they were resolved. This section is intentionally included because understanding failure modes is as important as the happy path.

---

## 1. Port 53 Conflict on the Host

**Problem**: When first running the Docker container, port 53 was already in use on the Ubuntu host. The container failed to start with a bind error.

**Cause**: Ubuntu's `systemd-resolved` service runs a local DNS stub listener on `127.0.0.53:53` by default.

**Resolution**:
Disabled the stub listener in `systemd-resolved` without fully disabling the service:

```bash
sudo nano /etc/systemd/resolved.conf
# Set: DNSStubListener=no

sudo systemctl restart systemd-resolved
```

Then symlinked the resolv.conf to restore DNS on the host:

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

**Takeaway**: Understanding what services are already running on a host before deploying containers is critical. `ss -tulpn | grep :53` is now a go-to first check.

---

## 2. DNS Broke Entirely When the Laptop Was Off

**Problem**: When the IdeaPad was shut down or went to sleep, every device on the network lost internet access because they couldn't resolve DNS.

**Cause**: The router was configured with Pi-hole as the **only** DNS server. No fallback meant no DNS.

**Resolution**:
- Disabled sleep/suspend on the IdeaPad (`sudo systemctl mask sleep.target suspend.target`)
- Set the Docker container to `restart: unless-stopped` to recover from crashes automatically
- Accepted the tradeoff of no secondary DNS to keep filtering enforced; documented this as a known limitation

**Takeaway**: High availability matters even in homelab setups. A single point of failure in DNS takes down the whole network. In a production context, a secondary Pi-hole instance would be the right solution.

---

## 3. Some Devices Ignored the Router's DNS Settings

**Problem**: A few devices (notably an Android phone with Private DNS enabled) were bypassing Pi-hole entirely.

**Cause**: Android 9+ supports **DNS-over-TLS (DoT)** with a "Private DNS" feature that overrides the router-assigned DNS server and uses a hardcoded secure resolver.

**Resolution**:
Disabled Private DNS on the affected device (Settings → Network → Private DNS → Off).

**Takeaway**: DNS-over-HTTPS and DNS-over-TLS are increasingly common and are designed to bypass traditional DNS interception. A truly robust network-level filtering setup would require firewall rules to block outbound port 853 (DoT) and intercept DoH traffic — noted as a future improvement.

---

## 4. Pi-hole Was Blocking Legitimate Services

**Problem**: After adding several community blocklists, some legitimate services started failing — including a streaming service and a software update endpoint.

**Cause**: Overly aggressive blocklists include domains that are also used by legitimate services.

**Resolution**:
Used the Pi-hole **Query Log** to identify which domains were being blocked, then added specific domains to the whitelist. Also reviewed and removed the most aggressive blocklists that caused the most false positives.

**Takeaway**: More blocklists ≠ better. Each list needs to be evaluated for false positive rate. Starting with a conservative baseline and adding incrementally is better than adding everything at once.

---

## What I'd Do Differently

- **Use environment variables** for Docker secrets instead of hardcoding the web password in `docker-compose.yml`
- **Set up a second Pi-hole instance** (e.g., on a Raspberry Pi) as a secondary DNS for redundancy
- **Implement DNS-over-TLS** on Pi-hole itself so that upstream queries to `1.1.1.1` are encrypted in transit
- **Add monitoring** — a simple uptime check or Grafana dashboard to track Pi-hole's health and query statistics over time
- **Document the router config** earlier in the process — took longer than expected to find the right settings on the specific router model in use
