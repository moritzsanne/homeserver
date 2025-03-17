# Home Server Setup

## Problem Statement
Need for a secure, reliable home server that segregates VPN-protected services from standard services, while maintaining easy access to both via a unified interface. Must handle torrenting, media services, and various utilities without exposing sensitive services to potential threats.

## Hardware
**GMKtec N150 Mini PC**
- Intel N100/N105 Alder Lake-N processor
- 8GB DDR4 RAM
- 256GB SSD storage
- Intel UHD Graphics with hardware acceleration capability
- 2.5Gb Ethernet connectivity

## OS Selection
**Ubuntu Server LTS** chosen for its reliability, Docker support, and hardware compatibility. 

Custom kernel updates were required to enable hardware acceleration for the N150 GPU.

## Docker & Portainer
Docker provides containerization for services, with Portainer offering easy management through a web interface. This combination enables efficient deployment and monitoring of multiple services with minimal overhead.

All service deployments are managed through Portainer's "Stacks" feature using Docker Compose files, providing version-controlled, reproducible configurations for each service group.

## Networking Setup
The network architecture uses two distinct network segments:
1. **Non-VPN Network (`non_vpn`)**: For services requiring standard internet access
2. **VPN Network Stack (`vpn_stack`)**: For privacy-sensitive services requiring tunneled access through Gluetun VPN container. Services connect to this network using `network_mode: service:gluetun` to share the VPN container's networking stack.

A single main reverse proxy (NGINX) on ports 80/443 serves as the entry point for all services, while a secondary proxy inside the VPN stack handles routing within the tunneled environment.

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
