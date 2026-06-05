# Setup Guide: Pi-hole on Docker (Ubuntu)

## Prerequisites

- A machine running **Ubuntu Linux** (this project used a Lenovo IdeaPad)
- Connected to your home LAN via Ethernet or Wi-Fi
- Basic familiarity with the Linux terminal
- Access to your router's admin panel

---

## Step 1 — Prepare the Host Machine

Update the system and install essential tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git net-tools -y
```

Verify your machine's current LAN IP address (you'll need this later):

```bash
ip addr show
# or
hostname -I
```

---

## Step 2 — Set a Static IP (via Router DHCP Reservation)

Rather than setting a static IP in the OS (which can cause conflicts), reserve the machine's IP in your **router's DHCP settings**:

1. Log into your router admin panel (typically `192.168.1.1` or `192.168.0.1`)
2. Find **DHCP Reservations** or **Address Reservation**
3. Bind the IdeaPad's MAC address to a fixed IP (e.g., `192.168.1.x`)
4. Save and reboot the router

This ensures the laptop always gets the same IP, making it a reliable DNS target.

---

## Step 3 — Install Docker

```bash
# Remove any old Docker versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install Docker using the official convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to the docker group (avoids needing sudo for docker commands)
sudo usermod -aG docker $USER

# Log out and back in, then verify installation
docker --version
```

---

## Step 4 — Deploy Pi-hole with Docker Compose

Create a project directory:

```bash
mkdir ~/pihole && cd ~/pihole
```

Create the `docker-compose.yml` file:

```yaml
version: "3"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
    environment:
      TZ: 'Asia/Manila'           # Set your timezone
      WEBPASSWORD: 'changeme'     # Change this to a strong password
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    restart: unless-stopped
```

> ⚠️ **Security note**: Change `WEBPASSWORD` to a strong password. Never commit real passwords to a public repository, use environment variable files (`.env`) excluded via `.gitignore`.

Start the container:

```bash
docker compose up -d
```

Check that it's running:

```bash
docker ps
docker logs pihole
```

---

## Step 5 — Configure the Router to Use Pi-hole as DNS

1. Log into your router admin panel
2. Find **DNS Server settings** (usually under LAN or DHCP settings)
3. Set **Primary DNS** to the static IP of the IdeaPad
4. Leave **Secondary DNS blank** (a fallback DNS would bypass Pi-hole)
5. Save and restart the router

All devices on the network will now use Pi-hole for DNS resolution.

---

## Step 6 — Verify It's Working

From any device on the network, check DNS resolution:

```bash
# Linux/macOS
nslookup doubleclick.net

# Expected result for a blocked domain:
# Address: 0.0.0.0
```

You can also visit the Pi-hole admin dashboard at:

```
http://<your-laptop-ip>/admin
```

---

## Step 7 — Add Blocklists (Optional but Recommended)

In the Pi-hole admin dashboard:
1. Go to **Group Management → Adlists**
2. Add community blocklist URLs (see [`docs/blocklists.md`](blocklists.md))
3. Run **Tools → Update Gravity** to apply the new lists

---

## Keeping Pi-hole Updated

```bash
# Pull the latest Pi-hole image
docker compose pull

# Recreate the container with the new image
docker compose up -d
```
