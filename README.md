# Docker AmneziaWG

[![Docker Build](https://github.com/AYastrebov/docker-amneziawg/actions/workflows/docker-build.yml/badge.svg)](https://github.com/AYastrebov/docker-amneziawg/actions/workflows/docker-build.yml)
[![Docker Pulls](https://img.shields.io/docker/pulls/ghcr.io/ayastrebov/docker-amneziawg?logo=docker)](https://github.com/AYastrebov/docker-amneziawg/pkgs/container/docker-amneziawg)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/AYastrebov/docker-amneziawg)](https://github.com/AYastrebov/docker-amneziawg/releases)
[![License](https://img.shields.io/github/license/AYastrebov/docker-amneziawg)](LICENSE)

A Docker container for running AmneziaWG, a modified version of WireGuard that provides enhanced obfuscation capabilities to bypass DPI (Deep Packet Inspection) and censorship.

## Overview

This project provides a containerized solution for running AmneziaWG VPN server/client. It builds upon the latest AmneziaWG-go implementation and includes pre-compiled AmneziaWG tools for easy deployment.

### Features

- 🚀 Latest AmneziaWG-go implementation
- 🛠️ Pre-compiled AmneziaWG tools (v1.0.20250706)
- 🐳 Multi-stage Docker build for optimized image size
- 🔧 Easy configuration management
- 🔄 Graceful shutdown handling
- 📦 Docker Compose ready
- 🏥 Built-in health checks
- 🏗️ Multi-architecture support (amd64, arm64)

## Quick Start

### Prerequisites

- Docker
- Docker Compose (optional)
- **Recommended**: AmneziaWG kernel module for optimal performance

> **💡 Performance Tip**: For better performance and lower CPU usage, it's highly recommended to install the AmneziaWG kernel module on your host system. See the [Kernel Module Installation](#kernel-module-installation) section below.

### Using Pre-built Image (Recommended)

The easiest way to get started is using the pre-built image from GitHub Packages:

```bash
# Pull and run the latest image
docker run -d \
  --name amneziawg \
  --cap-add NET_ADMIN \
  --cap-add SYS_MODULE \
  --device /dev/net/tun \
  --sysctl net.ipv4.ip_forward=1 \
  --sysctl net.ipv4.conf.all.src_valid_mark=1 \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -v $(pwd)/awg0.conf:/etc/wireguard/awg0.conf \
  ghcr.io/ayastrebov/docker-amneziawg:latest awg0
```

### Using Docker Compose (Recommended)

1. Clone this repository:
```bash
git clone https://github.com/AYastrebov/docker-amneziawg.git
cd docker-amneziawg
```

2. Create your AmneziaWG configuration file:
```bash
# Create your configuration file
cp awg0.conf.example awg0.conf
# Edit the configuration file with your settings
nano awg0.conf
```

3. Build and run the container:
```bash
docker-compose up -d
```

**Note**: The docker-compose.yml currently references a locally built image. To use the pre-built GitHub Packages image, update the `image` field in docker-compose.yml to:
```yaml
image: ghcr.io/ayastrebov/docker-amneziawg:latest
```

### Using Docker directly

1. Use the pre-built image:
```bash
docker run -d \
  --name amneziawg \
  --cap-add NET_ADMIN \
  --cap-add SYS_MODULE \
  --device /dev/net/tun \
  --sysctl net.ipv4.ip_forward=1 \
  --sysctl net.ipv4.conf.all.src_valid_mark=1 \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -v $(pwd)/awg0.conf:/etc/wireguard/awg0.conf \
  ghcr.io/ayastrebov/docker-amneziawg:latest awg0
```

Or build locally:

1. Build the image:
```bash
docker build -t amneziawg-go .
```

2. Run the container:
```bash
docker run -d \
  --name amneziawg \
  --cap-add NET_ADMIN \
  --cap-add SYS_MODULE \
  --device /dev/net/tun \
  --sysctl net.ipv4.ip_forward=1 \
  --sysctl net.ipv4.conf.all.src_valid_mark=1 \
  --sysctl net.ipv6.conf.all.disable_ipv6=0 \
  -v $(pwd)/awg0.conf:/etc/wireguard/awg0.conf \
  amneziawg-go awg0
```

## Configuration

### AmneziaWG Configuration File

Create a configuration file named `awg0.conf` (or any name matching your interface) in the project directory. Here's an example:

```ini
[Interface]
PrivateKey = <your-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
# AmneziaWG specific parameters
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 86
S2 = 12
H1 = 1755269708
H2 = 2101520157
H3 = 1829552136
H4 = 2016351429

[Peer]
PublicKey = <peer-public-key>
AllowedIPs = 10.0.0.2/32
```

### Environment Variables

The container accepts the following parameter:

- **Interface name**: Pass as the first argument (default: `wg0`)

Example:
```bash
docker run ... amneziawg-go awg0
```

## Kernel Module Installation

For optimal performance and lower CPU usage, it's **highly recommended** to install the AmneziaWG kernel module on your host system before running the container.

### Why Install the Kernel Module?

- **Better Performance**: Kernel-space implementation is more efficient than userspace
- **Lower CPU Usage**: Reduces overhead compared to the Go userspace implementation
- **Native Integration**: Works seamlessly with existing WireGuard tooling

### Installation

Install the AmneziaWG kernel module from the official repository:

```bash
# Clone the kernel module repository
git clone https://github.com/amnezia-vpn/amneziawg-linux-kernel-module.git
cd amneziawg-linux-kernel-module

# Follow the installation instructions in the repository
# This typically involves:
# 1. Installing kernel headers for your distribution
# 2. Compiling and installing the module
# 3. Loading the module

# Example for Ubuntu/Debian:
sudo apt update
sudo apt install linux-headers-$(uname -r) build-essential
make
sudo make install
sudo modprobe amneziawg
```

### Verification

After installation, verify the module is loaded:

```bash
# Check if the module is loaded
lsmod | grep amneziawg

# The container will automatically use the kernel module if available
# You can verify this in the container logs
```

> **Note**: If the kernel module is not available, the container will fall back to the userspace Go implementation automatically.

## Docker Compose Configuration

The included `docker-compose.yml` provides the following configuration:

- **Image**: Uses locally built image by default (`amneziawg-go`)
  - To use the pre-built GitHub Packages image, change to: `ghcr.io/ayastrebov/docker-amneziawg:latest`
- **Capabilities**: `NET_ADMIN` and `SYS_MODULE` for network management
- **Sysctls**: IP forwarding and routing configurations
- **Devices**: Access to `/dev/net/tun` for tunnel interface
- **Volumes**: Mounts your configuration file
- **Restart policy**: `unless-stopped` for automatic restart

### Using Pre-built Image

To use the GitHub Packages image with Docker Compose, update your `docker-compose.yml`:

```yaml
services:
  amneziawg:
    image: ghcr.io/ayastrebov/docker-amneziawg:latest
    container_name: amneziawg
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
    devices:
      - /dev/net/tun
    volumes:
      - ./awg0.conf:/etc/wireguard/awg0.conf
    command: awg0
```

## File Structure

```
.
├── .github/
│   ├── workflows/
│   │   └── docker-build.yml    # CI/CD pipeline
│   ├── ISSUE_TEMPLATE/         # Issue templates
│   ├── pull_request_template.md # PR template
│   └── RELEASE_TEMPLATE.md     # Release template
├── Dockerfile                  # Multi-stage build for AmneziaWG
├── docker-compose.yml          # Docker Compose configuration
├── entrypoint.sh              # Container entrypoint script
├── awg0.conf.example          # Example configuration file
├── awg0.conf                  # Your AmneziaWG configuration (create this)
├── CHANGELOG.md               # Version history
├── CONTRIBUTING.md            # Contribution guidelines
├── LICENSE                    # MIT License
├── SECURITY.md               # Security policy
└── README.md                 # This file
```

## Building from Source

> **Note**: Pre-built images are available on GitHub Packages. Building from source is only necessary if you need to customize the build or contribute to the project.

The project uses GitHub Actions to automatically build and publish Docker images to GitHub Packages. You can use the pre-built images with:

```bash
docker pull ghcr.io/ayastrebov/docker-amneziawg:latest
```

### Manual Build

The Dockerfile uses a multi-stage build:

1. **Builder stage**: Compiles AmneziaWG-go from source
2. **Runtime stage**: Creates minimal Alpine-based image with pre-compiled tools

### Build Arguments

- `AWGTOOLS_RELEASE`: Version of AmneziaWG tools to download (default: "1.0.20250706")

Example with custom tools version:
```bash
docker build --build-arg AWGTOOLS_RELEASE=1.0.20250706 -t amneziawg-go .
```

## Usage Examples

### Server Configuration

For a server setup, your configuration might look like:

```ini
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
# AmneziaWG obfuscation parameters
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 86
S2 = 12
H1 = 1755269708
H2 = 2101520157
H3 = 1829552136
H4 = 2016351429

[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.0.2/32
```

### Client Configuration

For a client setup:

```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.0.0.2/24
DNS = 8.8.8.8, 8.8.4.4
# AmneziaWG obfuscation parameters (must match server)
Jc = 4
Jmin = 50
Jmax = 1000
S1 = 86
S2 = 12
H1 = 1755269708
H2 = 2101520157
H3 = 1829552136
H4 = 2016351429

[Peer]
PublicKey = <server-public-key>
Endpoint = <server-ip>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

## Commands

### Container Management

```bash
# Start the service
docker-compose up -d

# Stop the service
docker-compose down

# View logs
docker-compose logs -f

# Restart the service
docker-compose restart
```

### WireGuard Management

Inside the container, you can use standard WireGuard commands with `awg` prefix:

```bash
# Check interface status
docker exec amneziawg awg show

# Show interface configuration
docker exec amneziawg awg show awg0

# Check container health status
docker ps  # Look for "healthy" status
docker inspect amneziawg --format='{{.State.Health.Status}}'

# Manual interface management (if needed)
docker exec amneziawg awg-quick up /etc/wireguard/awg0.conf
docker exec amneziawg awg-quick down /etc/wireguard/awg0.conf
```

## Troubleshooting

### Common Issues

1. **Permission denied errors**: Ensure the container has the required capabilities and device access
2. **Configuration not found**: Verify the configuration file is mounted correctly
3. **Network issues**: Check that IP forwarding is enabled and firewall rules are correct
4. **Poor performance**: Consider installing the AmneziaWG kernel module for better performance

### Debugging

Enable debug output:
```bash
# View container logs
docker-compose logs -f amneziawg

# Check interface status
docker exec amneziawg ip addr show

# Test connectivity
docker exec amneziawg ping <peer-ip>
```

## Security Considerations

- Keep your private keys secure and never commit them to version control
- Use strong, randomly generated keys
- Regularly update the container image for security patches
- Consider using Docker secrets for sensitive configuration data

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is licensed under the terms specified in the LICENSE file.

## Acknowledgments

- [AmneziaVPN](https://github.com/amnezia-vpn) for the AmneziaWG implementation
- [WireGuard](https://www.wireguard.com/) for the original protocol
- Alpine Linux for providing a minimal base image

## Links

- [Project Repository](https://github.com/AYastrebov/docker-amneziawg)
- [Docker Images (GitHub Packages)](https://github.com/AYastrebov/docker-amneziawg/pkgs/container/docker-amneziawg)
- [AmneziaWG Kernel Module](https://github.com/amnezia-vpn/amneziawg-linux-kernel-module) - **Recommended for better performance**
- [AmneziaWG GitHub](https://github.com/amnezia-vpn/amneziawg-go)
- [AmneziaWG Tools](https://github.com/amnezia-vpn/amneziawg-tools)
- [Official WireGuard Documentation](https://www.wireguard.com/)
