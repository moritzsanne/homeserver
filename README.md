# HomeLab Server 🏠🖥️

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](https://docs.docker.com/compose/)
[![Portainer](https://img.shields.io/badge/Portainer-Ready-brightgreen)](https://www.portainer.io/)

A comprehensive, easy-to-deploy home server setup built entirely with Docker Compose. Perfect for media management, self-hosting applications, and expanding your home lab with minimal configuration.

## 📋 Features

- **Docker-based**: Everything runs in containers for easy maintenance and deployment
- **Dual network architecture**: Separate VPN and non-VPN networks for privacy and security
- **Easy expansion**: Simple process for adding new services
- **HTTPS support**: Automatic SSL certificate management with Let's Encrypt and DuckDNS
- **Privacy-focused**: VPN integration for services that need enhanced privacy
- **Low resource requirements**: Optimized for compact mini PCs and small form factor hardware

## 🖥️ Hardware Requirements

This setup has been tested and optimized for the following hardware:

**GMKtec N150 Mini PC**
- Intel N150 Alder Lake-N processor
- 8GB DDR4 RAM
- 256GB SSD storage (plus expandable M.2 slot)
- Intel UHD Graphics (with hardware acceleration support)
- 2.5Gb Ethernet connectivity

However, any system meeting these minimum requirements should work fine:
- 4GB RAM
- 128GB storage
- 1Gb network connection
- x86-64 CPU with virtualization support

## 🚀 Quick Start

### Prerequisites

1. Install Docker and Docker Compose on your system
2. Install [Portainer](https://docs.portainer.io/start/install) for easy container management
3. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/homelab-server.git
   cd homelab-server
   ```

### Deployment

1. Access your Portainer instance (typically at http://your-server-ip:9000)
2. Navigate to "Stacks" and click "Add stack"
3. Upload the docker-compose.yml file from this repository
4. Configure your environment variables (see [Configuration](#%EF%B8%8F-configuration) section)
5. Click "Deploy the stack"

## 🏗️ Architecture

### Network Design

This project uses a dual-network architecture:

![Network Architecture](https://raw.githubusercontent.com/yourusername/homelab-server/main/docs/images/network_diagram.png)

1. **Non-VPN Network** (`non_vpn`)
   - For services requiring standard internet access
   - Includes media servers, monitoring tools, and web applications
   
2. **VPN Network Stack**
   - For privacy-sensitive services (torrenting, downloading)
   - Services connect using `network_mode: service:gluetun`
   - All traffic tunneled through the Gluetun VPN container

### Included Services

| Service | Description | Network | Port | 
|---------|-------------|---------|------|
| NGINX Proxy Manager | Reverse proxy with SSL | non_vpn | 80, 443 |
| Portainer | Container management | non_vpn | 9000 |
| Plex | Media server | non_vpn | 32400 |
| Sonarr | TV show management | non_vpn | 8989 |
| Radarr | Movie management | non_vpn | 7878 |
| Gluetun | VPN client container | non_vpn + vpn | - |
| qBittorrent | Torrent client | vpn (via gluetun) | - |
| Prowlarr | Indexer management | non_vpn | 9696 |
| *...and more* | | | |

## ⚙️ Configuration

### Environment Variables

Create a `.env` file based on the provided `.env.example`:

```bash
# Basic settings
TZ=America/New_York
PUID=1000
PGID=1000

# Storage paths
CONFIG_DIR=/path/to/config
MEDIA_DIR=/path/to/media
DOWNLOADS_DIR=/path/to/downloads

# VPN settings (for Gluetun)
VPN_SERVICE_PROVIDER=mullvad
VPN_USER=your_vpn_username
VPN_PASSWORD=your_vpn_password
```

### SSL Certificates

SSL certificates are automatically managed through Let's Encrypt with DNS challenges via DuckDNS:

1. Register for a free subdomain at [DuckDNS](https://www.duckdns.org/)
2. Add your DuckDNS token and domain to the `.env` file:
   ```
   DUCKDNS_TOKEN=your_token
   DUCKDNS_DOMAIN=your_domain
   ```

## 🔧 Adding New Services

### For Standard Services (Non-VPN)

Add to your docker-compose.yml:

```yaml
your-service:
  image: your-service-image
  container_name: your-service
  restart: unless-stopped
  networks:
    - non_vpn
  environment:
    - TZ=${TZ}
    - PUID=${PUID}
    - PGID=${PGID}
  volumes:
    - ${CONFIG_DIR}/your-service:/config
```

### For Privacy-Sensitive Services (VPN)

Add to your docker-compose.yml:

```yaml
your-vpn-service:
  image: your-service-image
  container_name: your-vpn-service
  restart: unless-stopped
  network_mode: service:gluetun
  environment:
    - TZ=${TZ}
    - PUID=${PUID}
    - PGID=${PGID}
  volumes:
    - ${CONFIG_DIR}/your-vpn-service:/config
```

## 💾 Backup Strategy

### Using Duplicati

The stack includes Duplicati for automated backups:

1. Access Duplicati web interface at http://your-server-ip:8200
2. Configure backups for your configuration directories
3. Recommended backup destinations:
   - Local external drive
   - Cloud storage (Google Drive, OneDrive, etc.)
   - Network storage (NAS)

### Backup Content

At minimum, back up your configuration directory:
```
/path/to/config
```

### Retention Policy

Recommended retention settings:
- Daily backups for the past week
- Weekly backups for the past month
- Monthly backups for the past year

## 🔄 Upgrading

To update all containers to their latest versions:

```bash
# Via docker compose directly
docker compose pull
docker compose up -d

# Via Portainer
# Go to your stack and click "Pull and redeploy"
```

## 🛠️ Hardware Optimization

### Hardware Acceleration

For Intel GPUs (like in the GMKtec N150):

1. Enable hardware transcoding in Plex:
   - Add these to the Plex container in docker-compose.yml:
     ```yaml
     devices:
       - /dev/dri:/dev/dri
     ```

2. Configure Plex settings:
   - Enable hardware acceleration in Plex settings (Settings > Transcoder)
   - Select "Intel QuickSync Video" as the acceleration type

### Storage Expansion

For adding more storage:

1. **Internal expansion**:
   - Install an M.2 SSD in the available slot
   - Format and mount the drive in your system

2. **External expansion**:
   - Connect an external drive via USB
   - Add to fstab for automatic mounting at boot

## 🔍 Troubleshooting

### Common Issues

#### Cannot access services outside local network

- Check port forwarding on your router
- Verify NGINX Proxy Manager configuration
- Ensure DuckDNS is updating your IP correctly

#### VPN performance issues

- Try different VPN servers
- Check if your ISP throttles VPN connections
- Adjust Gluetun settings for better performance

#### Container fails to start

Check logs with:
```bash
docker logs container-name
```

## 📚 Resources and References

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Portainer Documentation](https://docs.portainer.io/)
- [Gluetun VPN Client](https://github.com/qdm12/gluetun)
- [NGINX Proxy Manager](https://nginxproxymanager.com/)
- [Let's Encrypt](https://letsencrypt.org/)
- [DuckDNS](https://www.duckdns.org/)

## 📃 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👏 Acknowledgements

- All the open-source projects that make this possible
- The homelab and selfhosted communities on Reddit
