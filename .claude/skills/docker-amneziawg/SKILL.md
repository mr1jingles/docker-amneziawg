---
name: docker-amneziawg
description: |
  Development skill for the docker-amneziawg project - an AmneziaWG VPN container with LinuxServer.io architecture. Use when working in the docker-amneziawg repository for: (1) Adding features or fixing bugs, (2) Modifying s6-overlay services, (3) Updating config generation, (4) Working with AmneziaWG obfuscation parameters, (5) Testing or building the Docker image. Triggers when working in a directory containing this project's structure (root/etc/s6-overlay, awg-related files).
---

# docker-amneziawg Development Guide

## Documentation Layout

| File | Audience | Purpose |
|------|----------|---------|
| `README.md` | End users | Setup, usage, parameters (LinuxServer-style) |
| `CONTEXT.md` | AI agents | Architecture, parameter deep-dives, troubleshooting, CI/CD |
| `CLAUDE.md` | Developers | Dev patterns, conventions, gotchas, build/test commands |

For architecture details, parameter constraints, or troubleshooting tables, read `CONTEXT.md`.
For AWG parameter implementation specifics, read [references/awg-parameters.md](references/awg-parameters.md).

## Project Overview

AmneziaWG Docker container built on LinuxServer.io base images with s6-overlay process supervision. Provides automatic VPN configuration generation with DPI-bypass obfuscation.

Two modes: **server** (set `PEERS` to auto-generate configs) and **client** (place `.conf` files in `/config/wg_confs/`).

## Project Structure

```
docker-amneziawg/
├── Dockerfile                    # Multi-stage build (go-builder, tools-builder, runtime)
├── docker-compose.yml            # Example configurations
├── CONTEXT.md                    # Technical reference for AI agents
├── root/
│   ├── app/
│   │   └── show-peer             # QR code display utility
│   ├── defaults/
│   │   ├── server.conf           # Server config template (eval+heredoc)
│   │   ├── peer.conf             # Peer config template (eval+heredoc)
│   │   └── Corefile              # CoreDNS default config
│   └── etc/s6-overlay/s6-rc.d/
│       ├── init-amneziawg-module/    # Kernel module detection (oneshot)
│       ├── init-amneziawg-confs/     # Config generation (oneshot)
│       ├── svc-coredns/              # CoreDNS service (longrun)
│       ├── svc-amneziawg/            # Tunnel service (oneshot up/down)
│       └── user/contents.d/          # Service registration (empty files)
└── .github/workflows/
    ├── docker-build.yml              # Main build pipeline (multi-arch)
    └── upstream-check.yml            # Daily upstream version check
```

## S6-Overlay Architecture

### Service Dependency Chain
```
init-amneziawg-module (oneshot) -> init-amneziawg-confs (oneshot) -> svc-coredns (longrun) -> svc-amneziawg (oneshot)
```

Key points:
- `svc-amneziawg` is a **oneshot** — tunnels stay up without a running process
- `svc-coredns` is a **longrun** — continuously serves DNS for peers
- Dependencies: empty files in `dependencies.d/`. Registration: empty files in `user/contents.d/`

### Script Requirements
- Shebang: `#!/usr/bin/with-contenv bash`
- Must be executable (`chmod +x`)
- Use `lsiown` for LinuxServer permission management

## Key Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PEERS` | - | Enables server mode. Number ("3") or names ("laptop,phone") |
| `SERVERURL` | auto | Server URL/IP for peer configs |
| `SERVERPORT` | 51820 | Port advertised to peers. Use <= 9999 if ISP blocks high UDP |
| `INTERNAL_SUBNET` | 10.13.13.0 | VPN subnet (.1 = server, .2+ = peers) |
| `PEERDNS` | auto | DNS for peers (auto = container's CoreDNS at subnet.1) |
| `LOG_CONFS` | true | Show QR codes in container logs |
| `AWG_VERSION` | 2.0 | Protocol version: 2.0 (full DPI evasion) or 1.5 (legacy, AmneziaVPN < 4.8.12.9) |

## AmneziaWG Obfuscation — Quick Reference

For detailed parameter docs, see [references/awg-parameters.md](references/awg-parameters.md) or `CONTEXT.md`.

| Param | Default | Key Constraint |
|-------|---------|----------------|
| `AWG_S1` | Random 15-150 | <= 1132, **S1+56 must not equal S2** |
| `AWG_S2` | Random 15-150 | <= 1188 |
| `AWG_S3` | Random 8-55 (2.0) / 0 (1.5) | <= 64 |
| `AWG_S4` | Random 4-27 (2.0) / 0 (1.5) | <= 32, **per-packet overhead — keep small** |
| `AWG_H1-H4` | Range (2.0) / int (1.5) | >= 5, all unique, non-overlapping |
| `AWG_I1-I5` | Auto QUIC Initial (2.0) / empty (1.5) | In `[Interface]` before `[Peer]` |

**Critical**: Server and all clients must use identical S1-S4, H1-H4, I1-I5 values. Jc/Jmin/Jmax may differ.

## Common Development Tasks

### Adding a New Environment Variable
1. Set default in `init-amneziawg-confs/run` main logic section
2. If persistent: add to `save_vars()` (as `ORIG_X`) AND the change detection `if` block
3. For AWG params: also add to `generate_awg_params()` save block AND `load_awg_params()` grep section
4. For config output: add to templates in `root/defaults/` (eval+heredoc), `append_awg_signatures()` (server conf), or `append_awg_signatures_to_interface()` (peer confs — inserts before `[Peer]` via awk)
5. Document in `docker-compose.yml` and `README.md`

### Testing Changes
```bash
docker build -t amneziawg-test .
docker run -d --name awg-test --cap-add NET_ADMIN \
  -e PEERS=2 -e SERVERURL=test.example.com \
  -v /tmp/awg-test:/config amneziawg-test
docker logs awg-test
docker exec awg-test cat /config/wg_confs/wg0.conf
docker exec awg-test cat /config/peer1/peer1.conf
docker rm -f awg-test
```

Tunnel startup fails without `--device /dev/net/tun` — expected in testing.

## Common Gotchas

| Issue | Solution |
|-------|----------|
| `local: can only be used in a function` | Remove `local` keyword from main script body |
| awg-quick not found in build | Copy from `src/wg-quick/linux.bash`, not compiled |
| Service not starting | Check: executable bit, shebang, registered in `user/contents.d/` |
| Exit code 137 | Normal — container was stopped (SIGKILL) |
| I1-I5 must be in `[Interface]`, not `[Peer]` | Use `append_awg_signatures_to_interface()` for peer confs |
| `cut -d= -f2` truncates I-params with `=` | Use `cut -d= -f2-` (tag syntax contains `=` signs) |
| Loading `awg_params` with `source` | Never — overrides Docker env vars. Use `grep`/`cut` with `${VAR:-fallback}` |
| Amnezia app shows AWG 1.5 instead of 2.0 | H1-H4 must use range format, not single integers |
| `SERVERPORT` mapping in Docker | Map as `SERVERPORT:51820/udp` — container always listens on 51820 internally |

## GitHub Actions Workflows

### docker-build.yml
- Push to `master`/`main` -> builds multi-arch and tags as `latest` + tools version
- Push `v*` tags -> semantic version tags (1.0.0, 1.0, 1)
- Pull requests -> single-platform smoke test (no push)
- `workflow_dispatch` accepts version overrides

### upstream-check.yml
- Daily at 06:00 UTC: compares Dockerfile ARG defaults against latest upstream releases
- Auto-updates Dockerfile and triggers build if new version found
- Has concurrency control and version format validation

Multi-arch: `linux/amd64`, `linux/arm64`
