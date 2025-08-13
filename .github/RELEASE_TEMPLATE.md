## What's Changed

### 🚀 Features
- 

### 🐛 Bug Fixes
- 

### 📚 Documentation
- 

### 🔧 Maintenance
- 

### 🔒 Security
- 

## Docker Images

This release includes multi-architecture Docker images available at:
```bash
docker pull ghcr.io/ayastrebov/docker-amneziawg:v{{VERSION}}
docker pull ghcr.io/ayastrebov/docker-amneziawg:latest
```

## Supported Architectures
- linux/amd64
- linux/arm64

## Installation

### Using Docker Compose
```bash
# Update docker-compose.yml to use the new version
image: ghcr.io/ayastrebov/docker-amneziawg:v{{VERSION}}

# Run the container
docker-compose up -d
```

### Using Docker directly
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
  ghcr.io/ayastrebov/docker-amneziawg:v{{VERSION}} awg0
```

## Verification
```bash
# Verify the image
docker image inspect ghcr.io/ayastrebov/docker-amneziawg:v{{VERSION}}

# Check container health
docker ps  # Should show "healthy" status after startup
```

**Full Changelog**: https://github.com/AYastrebov/docker-amneziawg/compare/{{PREVIOUS_TAG}}...v{{VERSION}}
