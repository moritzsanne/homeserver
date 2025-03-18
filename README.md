# Home Server Setup

## Goal
The goal here is to create reproducable and easy to maintan home server setup. Besides basic setup everything runs in docker compose. Its main duties are media server and media management, but I I want to easily spin up self hosted services in docker containers when the need arises.

## Hardware
**GMKtec N150 Mini PC**
- Intel N150 Alder Lake-N processor
- 8GB DDR4 RAM
- 256GB SSD storage + empty M2 slot
- Intel UHD Graphics with hardware acceleration capability
- 2.5Gb Ethernet connectivity

## OS Selection
**Ubuntu Server LTS** chosen for its reliability, Docker support, and hardware compatibility. 

Custom kernel updates were required to enable hardware acceleration for the N150 GPU.

## Docker & Portainer
- I choose to run portainer as a docker management front end.
## Networking Setup
The network architecture uses two distinct network segments:
1. **Non-VPN Network (`non_vpn`)**: For services requiring standard internet access
2. **VPN Network Stack (`vpn_stack`)**: For privacy-sensitive services requiring tunneled access through Gluetun VPN container. Services connect to this network using `network_mode: service:gluetun` to share the VPN container's networking stack.

A single main reverse proxy (NGINX) on ports 80/443 serves as the entry point for all services, while a secondary proxy inside the VPN stack handles routing within the tunneled environment.
This should ensure that I can enforce SSL with signed certifcates on all open ports on the host machine.

### Setting up HTTPS
SSL certificates managed through Let's Encrypt with auto-renewal for all services using DNS challenges via DuckDNS. This free subdomain service allows for automated certificate management without exposing services to the internet. Certificates point to the local server IP, with the main proxy handling certificate termination for all internal services.

### VPN Port Forwarding for qBittorrent
qBittorrent runs within the VPN network stack using `network_mode: service:gluetun`, ensuring all torrent traffic is tunneled through the VPN. Port forwarding is configured directly in the Gluetun container settings. An automated script retrieves the dynamically assigned port from Gluetun and updates qBittorrent's configuration via its API, ensuring optimal connectivity for seeding.

### Backups
**TODO**: Add Duplicati backup documentation, including:
- Configuration settings
- Backup schedules
- Retention policies
- Storage destinations
- Encryption approach

### Storage Expansion
*[To be completed based on your storage solution]*

## Adding Services

### Service Placement Guide
1. **For VPN-required services:**
   - Add to VPN stack using `network_mode: service:gluetun` in compose file
   - Examples: torrent clients, media downloaders

2. **For standard services:**
   - Add to `non_vpn` network in compose file
   - Examples: media servers, monitoring tools

3. **For DNS services:**
   - Use `network_mode: host`
   - Example: AdGuard Home

### Access Configuration
Configure the main NGINX proxy to route requests based on domain/path to appropriate services.
