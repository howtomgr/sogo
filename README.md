# sogo Installation Guide

sogo is a free and open-source groupware server. SOGo provides scalable groupware server with AJAX web interface

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
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: HTTP/CalDAV/CardDAV
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default sogo port)
  - Various service ports
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

# Install sogo
sudo dnf install -y sogo

# Enable and start service
sudo systemctl enable --now sogo

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
sogo --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install sogo
sudo apt install -y sogo

# Enable and start service
sudo systemctl enable --now sogo

# Configure firewall
sudo ufw allow 443

# Verify installation
sogo --version
```

### Arch Linux

```bash
# Install sogo
sudo pacman -S sogo

# Enable and start service
sudo systemctl enable --now sogo

# Verify installation
sogo --version
```

### Alpine Linux

```bash
# Install sogo
apk add --no-cache sogo

# Enable and start service
rc-update add sogo default
rc-service sogo start

# Verify installation
sogo --version
```

### openSUSE/SLES

```bash
# Install sogo
sudo zypper install -y sogo

# Enable and start service
sudo systemctl enable --now sogo

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
sogo --version
```

### macOS

```bash
# Using Homebrew
brew install sogo

# Start service
brew services start sogo

# Verify installation
sogo --version
```

### FreeBSD

```bash
# Using pkg
pkg install sogo

# Enable in rc.conf
echo 'sogo_enable="YES"' >> /etc/rc.conf

# Start service
service sogo start

# Verify installation
sogo --version
```

### Windows

```bash
# Using Chocolatey
choco install sogo

# Or using Scoop
scoop install sogo

# Verify installation
sogo --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/sogo

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
sogo --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable sogo

# Start service
sudo systemctl start sogo

# Stop service
sudo systemctl stop sogo

# Restart service
sudo systemctl restart sogo

# Check status
sudo systemctl status sogo

# View logs
sudo journalctl -u sogo -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add sogo default

# Start service
rc-service sogo start

# Stop service
rc-service sogo stop

# Restart service
rc-service sogo restart

# Check status
rc-service sogo status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'sogo_enable="YES"' >> /etc/rc.conf

# Start service
service sogo start

# Stop service
service sogo stop

# Restart service
service sogo restart

# Check status
service sogo status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start sogo
brew services stop sogo
brew services restart sogo

# Check status
brew services list | grep sogo
```

### Windows Service Manager

```powershell
# Start service
net start sogo

# Stop service
net stop sogo

# Using PowerShell
Start-Service sogo
Stop-Service sogo
Restart-Service sogo

# Check status
Get-Service sogo
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream sogo_backend {
    server 127.0.0.1:443;
}

server {
    listen 80;
    server_name sogo.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sogo.example.com;

    ssl_certificate /etc/ssl/certs/sogo.example.com.crt;
    ssl_certificate_key /etc/ssl/private/sogo.example.com.key;

    location / {
        proxy_pass http://sogo_backend;
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
    ServerName sogo.example.com
    Redirect permanent / https://sogo.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sogo.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/sogo.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/sogo.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend sogo_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/sogo.pem
    redirect scheme https if !{ ssl_fc }
    default_backend sogo_backend

backend sogo_backend
    balance roundrobin
    server sogo1 127.0.0.1:443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R sogo:sogo /etc/sogo
sudo chmod 750 /etc/sogo

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
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
sudo systemctl status sogo

# View logs
sudo journalctl -u sogo -f

# Monitor resource usage
top -p $(pgrep sogo)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/sogo"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/sogo-backup-$DATE.tar.gz" /etc/sogo /var/lib/sogo

echo "Backup completed: $BACKUP_DIR/sogo-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop sogo

# Restore from backup
tar -xzf /backup/sogo/sogo-backup-*.tar.gz -C /

# Start service
sudo systemctl start sogo
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u sogo -n 100
sudo tail -f /var/log/sogo/sogo.log

# Check configuration
sogo --version

# Check permissions
ls -la /etc/sogo
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443

# Test connectivity
telnet localhost 443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep sogo)

# Check disk I/O
iotop -p $(pgrep sogo)

# Check connections
ss -an | grep 443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  sogo:
    image: sogo:latest
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/sogo
      - ./data:/var/lib/sogo
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update sogo

# Debian/Ubuntu
sudo apt update && sudo apt upgrade sogo

# Arch Linux
sudo pacman -Syu sogo

# Alpine Linux
apk update && apk upgrade sogo

# openSUSE
sudo zypper update sogo

# FreeBSD
pkg update && pkg upgrade sogo

# Always backup before updates
tar -czf /backup/sogo-pre-update-$(date +%Y%m%d).tar.gz /etc/sogo

# Restart after updates
sudo systemctl restart sogo
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/sogo

# Clean old logs
find /var/log/sogo -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/sogo
```

## Additional Resources

- Official Documentation: https://docs.sogo.org/
- GitHub Repository: https://github.com/sogo/sogo
- Community Forum: https://forum.sogo.org/
- Best Practices Guide: https://docs.sogo.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
