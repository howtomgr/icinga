# icinga Installation Guide

icinga is a free and open-source monitoring system. Icinga provides modern monitoring with Nagios compatibility

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
  - Storage: 5GB for data
  - Network: HTTP/API access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5665 (default icinga port)
  - Web on 80/443
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

# Install icinga
sudo dnf install -y icinga

# Enable and start service
sudo systemctl enable --now icinga

# Configure firewall
sudo firewall-cmd --permanent --add-port=5665/tcp
sudo firewall-cmd --reload

# Verify installation
icinga --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install icinga
sudo apt install -y icinga

# Enable and start service
sudo systemctl enable --now icinga

# Configure firewall
sudo ufw allow 5665

# Verify installation
icinga --version
```

### Arch Linux

```bash
# Install icinga
sudo pacman -S icinga

# Enable and start service
sudo systemctl enable --now icinga

# Verify installation
icinga --version
```

### Alpine Linux

```bash
# Install icinga
apk add --no-cache icinga

# Enable and start service
rc-update add icinga default
rc-service icinga start

# Verify installation
icinga --version
```

### openSUSE/SLES

```bash
# Install icinga
sudo zypper install -y icinga

# Enable and start service
sudo systemctl enable --now icinga

# Configure firewall
sudo firewall-cmd --permanent --add-port=5665/tcp
sudo firewall-cmd --reload

# Verify installation
icinga --version
```

### macOS

```bash
# Using Homebrew
brew install icinga

# Start service
brew services start icinga

# Verify installation
icinga --version
```

### FreeBSD

```bash
# Using pkg
pkg install icinga

# Enable in rc.conf
echo 'icinga_enable="YES"' >> /etc/rc.conf

# Start service
service icinga start

# Verify installation
icinga --version
```

### Windows

```bash
# Using Chocolatey
choco install icinga

# Or using Scoop
scoop install icinga

# Verify installation
icinga --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/icinga

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
icinga --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable icinga

# Start service
sudo systemctl start icinga

# Stop service
sudo systemctl stop icinga

# Restart service
sudo systemctl restart icinga

# Check status
sudo systemctl status icinga

# View logs
sudo journalctl -u icinga -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add icinga default

# Start service
rc-service icinga start

# Stop service
rc-service icinga stop

# Restart service
rc-service icinga restart

# Check status
rc-service icinga status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'icinga_enable="YES"' >> /etc/rc.conf

# Start service
service icinga start

# Stop service
service icinga stop

# Restart service
service icinga restart

# Check status
service icinga status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start icinga
brew services stop icinga
brew services restart icinga

# Check status
brew services list | grep icinga
```

### Windows Service Manager

```powershell
# Start service
net start icinga

# Stop service
net stop icinga

# Using PowerShell
Start-Service icinga
Stop-Service icinga
Restart-Service icinga

# Check status
Get-Service icinga
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream icinga_backend {
    server 127.0.0.1:5665;
}

server {
    listen 80;
    server_name icinga.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name icinga.example.com;

    ssl_certificate /etc/ssl/certs/icinga.example.com.crt;
    ssl_certificate_key /etc/ssl/private/icinga.example.com.key;

    location / {
        proxy_pass http://icinga_backend;
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
    ServerName icinga.example.com
    Redirect permanent / https://icinga.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName icinga.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/icinga.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/icinga.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5665/
    ProxyPassReverse / http://127.0.0.1:5665/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend icinga_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/icinga.pem
    redirect scheme https if !{ ssl_fc }
    default_backend icinga_backend

backend icinga_backend
    balance roundrobin
    server icinga1 127.0.0.1:5665 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R icinga:icinga /etc/icinga
sudo chmod 750 /etc/icinga

# Configure firewall
sudo firewall-cmd --permanent --add-port=5665/tcp
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
sudo systemctl status icinga

# View logs
sudo journalctl -u icinga -f

# Monitor resource usage
top -p $(pgrep icinga)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/icinga"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/icinga-backup-$DATE.tar.gz" /etc/icinga /var/lib/icinga

echo "Backup completed: $BACKUP_DIR/icinga-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop icinga

# Restore from backup
tar -xzf /backup/icinga/icinga-backup-*.tar.gz -C /

# Start service
sudo systemctl start icinga
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u icinga -n 100
sudo tail -f /var/log/icinga/icinga.log

# Check configuration
icinga --version

# Check permissions
ls -la /etc/icinga
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5665

# Test connectivity
telnet localhost 5665

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep icinga)

# Check disk I/O
iotop -p $(pgrep icinga)

# Check connections
ss -an | grep 5665
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  icinga:
    image: icinga:latest
    ports:
      - "5665:5665"
    volumes:
      - ./config:/etc/icinga
      - ./data:/var/lib/icinga
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update icinga

# Debian/Ubuntu
sudo apt update && sudo apt upgrade icinga

# Arch Linux
sudo pacman -Syu icinga

# Alpine Linux
apk update && apk upgrade icinga

# openSUSE
sudo zypper update icinga

# FreeBSD
pkg update && pkg upgrade icinga

# Always backup before updates
tar -czf /backup/icinga-pre-update-$(date +%Y%m%d).tar.gz /etc/icinga

# Restart after updates
sudo systemctl restart icinga
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/icinga

# Clean old logs
find /var/log/icinga -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/icinga
```

## Additional Resources

- Official Documentation: https://docs.icinga.org/
- GitHub Repository: https://github.com/icinga/icinga
- Community Forum: https://forum.icinga.org/
- Best Practices Guide: https://docs.icinga.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
