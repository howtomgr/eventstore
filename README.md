# eventstore Installation Guide

eventstore is a free and open-source event sourcing database. EventStore is designed for event sourcing with built-in projections and subscriptions, serving as a specialized database for event-driven architectures

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
  - CPU: 2+ cores recommended
  - RAM: 4GB minimum (8GB+ recommended)
  - Storage: 10GB+ for events
  - Network: HTTP and TCP
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 2113 (default eventstore port)
  - Port 1113 for TCP
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

# Install eventstore
sudo dnf install -y eventstore

# Enable and start service
sudo systemctl enable --now eventstore

# Configure firewall
sudo firewall-cmd --permanent --add-port=2113/tcp
sudo firewall-cmd --reload

# Verify installation
eventstore --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install eventstore
sudo apt install -y eventstore

# Enable and start service
sudo systemctl enable --now eventstore

# Configure firewall
sudo ufw allow 2113

# Verify installation
eventstore --version
```

### Arch Linux

```bash
# Install eventstore
sudo pacman -S eventstore

# Enable and start service
sudo systemctl enable --now eventstore

# Verify installation
eventstore --version
```

### Alpine Linux

```bash
# Install eventstore
apk add --no-cache eventstore

# Enable and start service
rc-update add eventstore default
rc-service eventstore start

# Verify installation
eventstore --version
```

### openSUSE/SLES

```bash
# Install eventstore
sudo zypper install -y eventstore

# Enable and start service
sudo systemctl enable --now eventstore

# Configure firewall
sudo firewall-cmd --permanent --add-port=2113/tcp
sudo firewall-cmd --reload

# Verify installation
eventstore --version
```

### macOS

```bash
# Using Homebrew
brew install eventstore

# Start service
brew services start eventstore

# Verify installation
eventstore --version
```

### FreeBSD

```bash
# Using pkg
pkg install eventstore

# Enable in rc.conf
echo 'eventstore_enable="YES"' >> /etc/rc.conf

# Start service
service eventstore start

# Verify installation
eventstore --version
```

### Windows

```bash
# Using Chocolatey
choco install eventstore

# Or using Scoop
scoop install eventstore

# Verify installation
eventstore --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/eventstore

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
eventstore --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable eventstore

# Start service
sudo systemctl start eventstore

# Stop service
sudo systemctl stop eventstore

# Restart service
sudo systemctl restart eventstore

# Check status
sudo systemctl status eventstore

# View logs
sudo journalctl -u eventstore -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add eventstore default

# Start service
rc-service eventstore start

# Stop service
rc-service eventstore stop

# Restart service
rc-service eventstore restart

# Check status
rc-service eventstore status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'eventstore_enable="YES"' >> /etc/rc.conf

# Start service
service eventstore start

# Stop service
service eventstore stop

# Restart service
service eventstore restart

# Check status
service eventstore status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start eventstore
brew services stop eventstore
brew services restart eventstore

# Check status
brew services list | grep eventstore
```

### Windows Service Manager

```powershell
# Start service
net start eventstore

# Stop service
net stop eventstore

# Using PowerShell
Start-Service eventstore
Stop-Service eventstore
Restart-Service eventstore

# Check status
Get-Service eventstore
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream eventstore_backend {
    server 127.0.0.1:2113;
}

server {
    listen 80;
    server_name eventstore.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name eventstore.example.com;

    ssl_certificate /etc/ssl/certs/eventstore.example.com.crt;
    ssl_certificate_key /etc/ssl/private/eventstore.example.com.key;

    location / {
        proxy_pass http://eventstore_backend;
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
    ServerName eventstore.example.com
    Redirect permanent / https://eventstore.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName eventstore.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/eventstore.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/eventstore.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:2113/
    ProxyPassReverse / http://127.0.0.1:2113/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend eventstore_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/eventstore.pem
    redirect scheme https if !{ ssl_fc }
    default_backend eventstore_backend

backend eventstore_backend
    balance roundrobin
    server eventstore1 127.0.0.1:2113 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R eventstore:eventstore /etc/eventstore
sudo chmod 750 /etc/eventstore

# Configure firewall
sudo firewall-cmd --permanent --add-port=2113/tcp
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
sudo systemctl status eventstore

# View logs
sudo journalctl -u eventstore -f

# Monitor resource usage
top -p $(pgrep eventstore)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/eventstore"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/eventstore-backup-$DATE.tar.gz" /etc/eventstore /var/lib/eventstore

echo "Backup completed: $BACKUP_DIR/eventstore-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop eventstore

# Restore from backup
tar -xzf /backup/eventstore/eventstore-backup-*.tar.gz -C /

# Start service
sudo systemctl start eventstore
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u eventstore -n 100
sudo tail -f /var/log/eventstore/eventstore.log

# Check configuration
eventstore --version

# Check permissions
ls -la /etc/eventstore
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 2113

# Test connectivity
telnet localhost 2113

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep eventstore)

# Check disk I/O
iotop -p $(pgrep eventstore)

# Check connections
ss -an | grep 2113
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  eventstore:
    image: eventstore:latest
    ports:
      - "2113:2113"
    volumes:
      - ./config:/etc/eventstore
      - ./data:/var/lib/eventstore
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update eventstore

# Debian/Ubuntu
sudo apt update && sudo apt upgrade eventstore

# Arch Linux
sudo pacman -Syu eventstore

# Alpine Linux
apk update && apk upgrade eventstore

# openSUSE
sudo zypper update eventstore

# FreeBSD
pkg update && pkg upgrade eventstore

# Always backup before updates
tar -czf /backup/eventstore-pre-update-$(date +%Y%m%d).tar.gz /etc/eventstore

# Restart after updates
sudo systemctl restart eventstore
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/eventstore

# Clean old logs
find /var/log/eventstore -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/eventstore
```

## Additional Resources

- Official Documentation: https://docs.eventstore.org/
- GitHub Repository: https://github.com/eventstore/eventstore
- Community Forum: https://forum.eventstore.org/
- Best Practices Guide: https://docs.eventstore.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
