# Docker AmneziaWG

[![Docker Build](https://github.com/AYastrebov/docker-amneziawg/actions/workflows/docker-build.yml/badge.svg)](https://github.com/AYastrebov/docker-amneziawg/actions/workflows/docker-build.yml)
[![GitHub Container Registry](https://img.shields.io/badge/ghcr.io-docker--amneziawg-blue?logo=docker)](https://github.com/AYastrebov/docker-amneziawg/pkgs/container/docker-amneziawg)
[![GitHub release](https://img.shields.io/github/v/release/AYastrebov/docker-amneziawg)](https://github.com/AYastrebov/docker-amneziawg/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Docker container for running AmneziaWG VPN with automatic configuration generation, peer management, and QR code support. Built on LinuxServer.io base images with s6-overlay process supervision.

## Features

- **Automatic Configuration**: Generate server and peer configs from environment variables
- **QR Code Support**: Display peer configs as QR codes for easy mobile setup
- **AmneziaWG Obfuscation**: Built-in DPI bypass with random or custom obfuscation parameters
- **Multi-Peer Management**: Support for numbered or named peers (e.g., `laptop,phone,tablet`)
- **s6-overlay Supervision**: Reliable process management with graceful shutdown
- **Dual Mode**: Server mode (auto-generate) or Client mode (manual configs)
- **Multi-Architecture**: Supports `linux/amd64` and `linux/arm64`

## Quick Start

### Server Mode (Recommended)

Create a VPN server with automatic peer configuration:

```bash
docker run -d \
  --name amneziawg \
  --cap-add NET_ADMIN \
  --cap-add SYS_MODULE \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e SERVERURL=vpn.example.com \
  -e PEERS=3 \
  -p 51820:51820/udp \
  -v ./config:/config \
  --sysctl net.ipv4.ip_forward=1 \
  --sysctl net.ipv4.conf.all.src_valid_mark=1 \
  --restart unless-stopped \
  ghcr.io/ayastrebov/docker-amneziawg:latest
```

View QR codes for peers:

```bash
docker exec amneziawg /app/show-peer 1 2 3
```

### Client Mode

Use pre-existing configuration files:

```bash
# Place your config in ./config/wg_confs/wg0.conf
docker run -d \
  --name amneziawg \
  --cap-add NET_ADMIN \
  --cap-add SYS_MODULE \
  -v ./config:/config \
  --sysctl net.ipv4.ip_forward=1 \
  --sysctl net.ipv4.conf.all.src_valid_mark=1 \
  --restart unless-stopped \
  ghcr.io/ayastrebov/docker-amneziawg:latest
```

### Docker Compose

```yaml
services:
  amneziawg:
    image: ghcr.io/ayastrebov/docker-amneziawg:latest
    container_name: amneziawg
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=vpn.example.com
      - SERVERPORT=51820
      - PEERS=laptop,phone,tablet
      - PEERDNS=8.8.8.8, 8.8.4.4
      - INTERNAL_SUBNET=10.13.13.0
      - ALLOWEDIPS=0.0.0.0/0, ::/0
      - LOG_CONFS=true
    volumes:
      - ./config:/config
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

## Environment Variables

### Server Mode (when `PEERS` is set)

| Variable | Default | Description |
|----------|---------|-------------|
| `PEERS` | - | Number or comma-separated names (enables server mode) |
| `SERVERURL` | `auto` | External server URL/IP (`auto` to detect) |
| `SERVERPORT` | `51820` | Listen port |
| `INTERNAL_SUBNET` | `10.13.13.0` | VPN subnet (peers get .2, .3, etc.) |
| `PEERDNS` | `auto` | DNS for peers (`auto` = 8.8.8.8, 8.8.4.4) |
| `ALLOWEDIPS` | `0.0.0.0/0, ::/0` | Peer allowed IPs |
| `LOG_CONFS` | `true` | Show QR codes in container logs |
| `INTERFACE` | `wg0` | Interface name |

### AmneziaWG Obfuscation

All obfuscation parameters are optional. If not set, random values are generated for anti-DPI protection.

| Variable | Description |
|----------|-------------|
| `AWG_JC` | Junk packet count (recommended: 3-8) |
| `AWG_JMIN` | Minimum junk packet size in bytes |
| `AWG_JMAX` | Maximum junk packet size in bytes |
| `AWG_S1` | Init packet padding size |
| `AWG_S2` | Response packet padding size |
| `AWG_S3` | Cookie message padding size (default: 0) |
| `AWG_S4` | Transport packet padding size (default: 0) |
| `AWG_H1` | Header obfuscation value 1 |
| `AWG_H2` | Header obfuscation value 2 |
| `AWG_H3` | Header obfuscation value 3 |
| `AWG_H4` | Header obfuscation value 4 |

### LinuxServer Standard

| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | `1000` | User ID for file ownership |
| `PGID` | `1000` | Group ID for file ownership |
| `TZ` | `Etc/UTC` | Timezone |

## Configuration

### Volume Structure

```
./config/
├── wg_confs/           # WireGuard config files (auto-generated or manual)
│   └── wg0.conf        # Server config (interface)
├── server/             # Server keys and params (auto-generated)
│   ├── privatekey-server
│   ├── publickey-server
│   └── awg_params      # Saved AWG obfuscation parameters
├── peer1/              # Peer configs (auto-generated)
│   ├── peer1.conf
│   ├── peer1.png       # QR code image
│   ├── privatekey-peer1
│   ├── publickey-peer1
│   └── presharedkey-peer1
└── laptop/             # Named peer example
    ├── laptop.conf
    └── laptop.png
```

### Manual Configuration (Client Mode)

Place your configuration files in `./config/wg_confs/`:

```ini
# ./config/wg_confs/wg0.conf
[Interface]
PrivateKey = <your-private-key>
Address = 10.0.0.2/32
DNS = 8.8.8.8
# AmneziaWG parameters (must match server)
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 86
S2 = 12
S3 = 0
S4 = 0
H1 = 1755269708
H2 = 2101520157
H3 = 1829552136
H4 = 2016351429

[Peer]
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Commands

### Show Peer QR Codes

```bash
# By number
docker exec amneziawg /app/show-peer 1 2 3

# By name
docker exec amneziawg /app/show-peer laptop phone tablet

# All peers
docker exec amneziawg /app/show-peer all
```

### Check Status

```bash
# Interface status
docker exec amneziawg awg show

# Container logs
docker logs amneziawg

# Health check
docker inspect amneziawg --format='{{.State.Health.Status}}'
```

### Manual Interface Control

```bash
# Bring down interface
docker exec amneziawg awg-quick down /config/wg_confs/wg0.conf

# Bring up interface
docker exec amneziawg awg-quick up /config/wg_confs/wg0.conf
```

## Migration from Previous Version

If you're upgrading from the previous simple entrypoint version:

### Automatic Migration

The container automatically migrates legacy configs:
- `/config/awg0.conf` → `/config/wg_confs/wg0.conf`
- `/config/wg0.conf` → `/config/wg_confs/wg0.conf`

### Manual Migration

1. Update your volume mount:
   ```yaml
   # Old
   volumes:
     - ./awg0.conf:/etc/wireguard/awg0.conf

   # New
   volumes:
     - ./config:/config
   ```

2. Move your config file:
   ```bash
   mkdir -p ./config/wg_confs
   mv ./awg0.conf ./config/wg_confs/wg0.conf
   ```

3. Remove the `command:` line from your docker-compose.yml (no longer needed)

## Kernel Module

For optimal performance, install the AmneziaWG kernel module on your host:

```bash
git clone https://github.com/amnezia-vpn/amneziawg-linux-kernel-module.git
cd amneziawg-linux-kernel-module
sudo apt install linux-headers-$(uname -r) build-essential  # Debian/Ubuntu
make && sudo make install
sudo modprobe amneziawg
```

The container automatically detects and uses:
1. `amneziawg` kernel module (preferred)
2. `wireguard` kernel module (compatibility mode)
3. `amneziawg-go` userspace (fallback)

## Building

### Local Build

```bash
docker build -t amneziawg .
```

### Multi-Architecture Build

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t amneziawg .
```

## Project Structure

```
docker-amneziawg/
├── Dockerfile                              # Multi-stage build
├── docker-compose.yml                      # Example configuration
├── root/
│   ├── app/
│   │   └── show-peer                       # QR code display utility
│   ├── defaults/
│   │   ├── server.conf                     # Server template
│   │   └── peer.conf                       # Peer template
│   └── etc/s6-overlay/s6-rc.d/
│       ├── init-amneziawg-module/          # Kernel module validation
│       ├── init-amneziawg-confs/           # Config generation
│       └── svc-amneziawg/                  # Tunnel service
├── awg0.conf.example                       # Example config
└── README.md
```

## Troubleshooting

### No configuration files found

Ensure you either:
- Set `PEERS` environment variable for server mode
- Place `.conf` files in `./config/wg_confs/`

### Permission denied

Check that capabilities are set:
```yaml
cap_add:
  - NET_ADMIN
  - SYS_MODULE
```

### Tunnel fails to start

Check container logs:
```bash
docker logs amneziawg
```

Verify sysctl settings:
```yaml
sysctls:
  - net.ipv4.ip_forward=1
  - net.ipv4.conf.all.src_valid_mark=1
```

### QR code not displaying

Ensure `LOG_CONFS=true` is set, or use:
```bash
docker exec amneziawg /app/show-peer all
```

## Links

- [GitHub Repository](https://github.com/AYastrebov/docker-amneziawg)
- [Docker Images](https://github.com/AYastrebov/docker-amneziawg/pkgs/container/docker-amneziawg)
- [AmneziaWG Kernel Module](https://github.com/amnezia-vpn/amneziawg-linux-kernel-module)
- [AmneziaWG-go](https://github.com/amnezia-vpn/amneziawg-go)
- [AmneziaWG Tools](https://github.com/amnezia-vpn/amneziawg-tools)
- [LinuxServer.io](https://www.linuxserver.io/)

## License

MIT License - see [LICENSE](LICENSE) file.
