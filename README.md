# cadence Installation Guide

cadence is a free and open-source workflow orchestration. Uber Cadence provides fault-tolerant stateful code platform

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 4+ cores
  - RAM: 4GB minimum
  - Storage: 20GB for history
  - Network: Thrift/gRPC
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 7933 (default cadence port)
  - Web UI on 8088
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install cadence
sudo dnf install -y cadence

# Enable and start service
sudo systemctl enable --now cadence

# Configure firewall
sudo firewall-cmd --permanent --add-port=7933/tcp
sudo firewall-cmd --reload

# Verify installation
cadence --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install cadence
sudo apt install -y cadence

# Enable and start service
sudo systemctl enable --now cadence

# Configure firewall
sudo ufw allow 7933

# Verify installation
cadence --version
```

### Arch Linux

```bash
# Install cadence
sudo pacman -S cadence

# Enable and start service
sudo systemctl enable --now cadence

# Verify installation
cadence --version
```

### Alpine Linux

```bash
# Install cadence
apk add --no-cache cadence

# Enable and start service
rc-update add cadence default
rc-service cadence start

# Verify installation
cadence --version
```

### openSUSE/SLES

```bash
# Install cadence
sudo zypper install -y cadence

# Enable and start service
sudo systemctl enable --now cadence

# Configure firewall
sudo firewall-cmd --permanent --add-port=7933/tcp
sudo firewall-cmd --reload

# Verify installation
cadence --version
```

### macOS

```bash
# Using Homebrew
brew install cadence

# Start service
brew services start cadence

# Verify installation
cadence --version
```

### FreeBSD

```bash
# Using pkg
pkg install cadence

# Enable in rc.conf
echo 'cadence_enable="YES"' >> /etc/rc.conf

# Start service
service cadence start

# Verify installation
cadence --version
```

### Windows

```bash
# Using Chocolatey
choco install cadence

# Or using Scoop
scoop install cadence

# Verify installation
cadence --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/cadence

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
cadence --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable cadence

# Start service
sudo systemctl start cadence

# Stop service
sudo systemctl stop cadence

# Restart service
sudo systemctl restart cadence

# Check status
sudo systemctl status cadence

# View logs
sudo journalctl -u cadence -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add cadence default

# Start service
rc-service cadence start

# Stop service
rc-service cadence stop

# Restart service
rc-service cadence restart

# Check status
rc-service cadence status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'cadence_enable="YES"' >> /etc/rc.conf

# Start service
service cadence start

# Stop service
service cadence stop

# Restart service
service cadence restart

# Check status
service cadence status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start cadence
brew services stop cadence
brew services restart cadence

# Check status
brew services list | grep cadence
```

### Windows Service Manager

```powershell
# Start service
net start cadence

# Stop service
net stop cadence

# Using PowerShell
Start-Service cadence
Stop-Service cadence
Restart-Service cadence

# Check status
Get-Service cadence
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream cadence_backend {
    server 127.0.0.1:7933;
}

server {
    listen 80;
    server_name cadence.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cadence.example.com;

    ssl_certificate /etc/ssl/certs/cadence.example.com.crt;
    ssl_certificate_key /etc/ssl/private/cadence.example.com.key;

    location / {
        proxy_pass http://cadence_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName cadence.example.com
    Redirect permanent / https://cadence.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName cadence.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cadence.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/cadence.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:7933/
    ProxyPassReverse / http://127.0.0.1:7933/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend cadence_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/cadence.pem
    redirect scheme https if !{ ssl_fc }
    default_backend cadence_backend

backend cadence_backend
    balance roundrobin
    server cadence1 127.0.0.1:7933 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R cadence:cadence /etc/cadence
sudo chmod 750 /etc/cadence

# Configure firewall
sudo firewall-cmd --permanent --add-port=7933/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status cadence

# View logs
sudo journalctl -u cadence -f

# Monitor resource usage
top -p $(pgrep cadence)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/cadence"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/cadence-backup-$DATE.tar.gz" /etc/cadence /var/lib/cadence

echo "Backup completed: $BACKUP_DIR/cadence-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop cadence

# Restore from backup
tar -xzf /backup/cadence/cadence-backup-*.tar.gz -C /

# Start service
sudo systemctl start cadence
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u cadence -n 100
sudo tail -f /var/log/cadence/cadence.log

# Check configuration
cadence --version

# Check permissions
ls -la /etc/cadence
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 7933

# Test connectivity
telnet localhost 7933

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep cadence)

# Check disk I/O
iotop -p $(pgrep cadence)

# Check connections
ss -an | grep 7933
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  cadence:
    image: cadence:latest
    ports:
      - "7933:7933"
    volumes:
      - ./config:/etc/cadence
      - ./data:/var/lib/cadence
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update cadence

# Debian/Ubuntu
sudo apt update && sudo apt upgrade cadence

# Arch Linux
sudo pacman -Syu cadence

# Alpine Linux
apk update && apk upgrade cadence

# openSUSE
sudo zypper update cadence

# FreeBSD
pkg update && pkg upgrade cadence

# Always backup before updates
tar -czf /backup/cadence-pre-update-$(date +%Y%m%d).tar.gz /etc/cadence

# Restart after updates
sudo systemctl restart cadence
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/cadence

# Clean old logs
find /var/log/cadence -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/cadence
```

## Additional Resources

- Official Documentation: https://docs.cadence.org/
- GitHub Repository: https://github.com/cadence/cadence
- Community Forum: https://forum.cadence.org/
- Best Practices Guide: https://docs.cadence.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
