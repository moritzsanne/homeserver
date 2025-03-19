# Home Server Setup🏠🖥️

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![Portainer](https://img.shields.io/badge/Portainer-Ready-brightgreen)](https://www.portainer.io/)

A comprehensive, easy-to-deploy home server setup built entirely with Docker Compose. Perfect for media management, self-hosting applications, and expanding your home lab with minimal configuration.

## 📋 Features

- **Docker-based**: Everything runs in containers for easy maintenance and deployment
- **Easy expansion**: Simple process for adding new services
- **HTTPS support**: Automatic SSL certificate management with Let's Encrypt and DuckDNS
- **Privacy-focused**: VPN integration for services that need enhanced privacy
- **Low resource requirements**: Optimized for compact mini PCs and small form factor hardware

## Project Structure
The project is organized into the following directories:

home-server-core/: Core services that are essential for the server to function
   - Nginx Proxy Manager: Reverse proxy for routing traffic to different services
   - Duplicati: Backup solution for data protection
media-server/
   - Jellyfin: Media server for streaming movies, TV shows, and music
   - Sonarr: TV show management and downloading
   - Radarr: Movie management and downloading
   - Prawlarr: Indexer for Sonarr and Radarr
   - Flaresolverr: Proxy for passing Cloudflare DDoS protection
   - qbittorrent: Torrent client for downloading files
   - gluetun: VPN container for routing traffic through a VPN

## 🖥️ Hardware Requirements

This setup has been tested and optimized for the following hardware:

**GMKtec N150 Mini PC**
- Intel N150 Alder Lake-N processor
- 8GB DDR4 RAM
- 256GB SSD storage (plus expandable M.2 slot)
- Intel UHD Graphics (with hardware acceleration support)
- 2.5Gb Ethernet connectivity
## 🚀 Quick Start

### Prerequisites
This setup is based on Ubuntu Server LTS. Before you begin, ensure you have the following:
1. Install Docker and Docker Compose on your system
2. Install [Portainer](https://docs.portainer.io/start/install) for easy container management
3. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/homelab-server.git
   cd homelab-server
   ```
4. If you want to use your intel GPU for transcoding, ensure your current kernel version supports your GPU.

### Deployment

1. Access your Portainer instance (typically at http://your-server-ip:9000)
2. Navigate to "Stacks" and click "Add stack"
3. Upload the docker-compose.yml file from this repository
4. Configure your environment variables or load example.env from the stack subdirectory and fill in the required values
5. Click "Deploy the stack"

## 🏗️ Architecture

### Network Design

This project uses a dual-network architecture:

1. Nginx Proxy Manager
   - Acts as a reverse proxy for all services
   - Handles SSL termination and routing
   - You can easily setup new services with custom domains through the userinterface at http://your-server-ip:81
2. **VPN Network Stack**
   - For privacy-sensitive services (torrenting, downloading)
   - Services connect using `network_mode: service:gluetun`
   - All traffic tunneled through the Gluetun VPN container

### SSL Certificates

SSL certificates are automatically managed through Let's Encrypt with DNS challenges via DuckDNS:

1. Register for a free subdomain at [DuckDNS](https://www.duckdns.org/)
2. Point your domain to your private IP address, no need to expose your server to the internet
3. Set up dns-01 challenge in Nginx Proxy Manager for automatic certificate renewal using your DuckDNS token

### Backup Content
At minimum, back up your configuration directory:
```
/path/to/config
```