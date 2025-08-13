# Security Policy

## Supported Versions

We support the latest version of this Docker image. Security updates are applied to the `latest` tag and new releases.

| Version | Supported          |
| ------- | ------------------ |
| latest  | ✅ Yes             |
| < 1.0   | ❌ No              |

## Reporting a Vulnerability

If you discover a security vulnerability, please follow these steps:

1. **Do NOT** open a public issue
2. Include the following information:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

## Security Best Practices

When using this Docker image:

### Configuration Security
- **Never** commit configuration files with real keys to version control
- Use strong, randomly generated private keys
- Regularly rotate keys
- Limit `AllowedIPs` to necessary ranges only

### Container Security
- Run containers with minimal required privileges
- Use Docker secrets for sensitive configuration
- Regularly update the container image
- Monitor container logs for suspicious activity

### Network Security
- Use firewall rules to restrict access to WireGuard ports
- Consider using non-standard ports
- Enable logging for connection monitoring
- Use strong authentication for server access

### Host Security
- Keep the Docker host system updated
- Use container runtime security tools
- Implement proper backup strategies for configurations
- Monitor system resources and network traffic

## Security Features

This image includes several security enhancements:

- **Minimal Attack Surface**: Based on Alpine Linux with minimal packages
- **Non-Root Execution**: Runs with appropriate user privileges
- **Static Binary**: Uses statically compiled Go binary to avoid library vulnerabilities
- **Graceful Shutdown**: Properly handles termination signals
- **AmneziaWG Obfuscation**: Built-in traffic obfuscation to evade detection

## Vulnerability Response

- Security issues will be addressed with high priority
- Fixes will be released as soon as possible
- Security advisories will be published for significant vulnerabilities
- Users will be notified through GitHub releases and repository updates

## Security Updates

Stay informed about security updates:

1. Watch this repository for releases
2. Subscribe to GitHub security advisories
3. Follow the project's release notes
4. Check for updates regularly using `docker pull`

## Responsible Disclosure

We appreciate security researchers who help keep our users safe. If you report a vulnerability responsibly, we will:

- Work with you to understand and resolve the issue
- Provide credit for the discovery (if desired)
- Keep you informed of our progress

Thank you for helping keep Docker AmneziaWG secure! 🔒
