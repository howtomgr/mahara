# mahara Installation Guide

mahara is a free and open-source ePortfolio system. Mahara provides learner-centered ePortfolio system

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
  - CPU: 1 core minimum
  - RAM: 1GB minimum
  - Storage: 10GB for portfolios
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default mahara port)
  - None
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

# Install mahara
sudo dnf install -y mahara

# Enable and start service
sudo systemctl enable --now mahara

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mahara --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install mahara
sudo apt install -y mahara

# Enable and start service
sudo systemctl enable --now mahara

# Configure firewall
sudo ufw allow 80

# Verify installation
mahara --version
```

### Arch Linux

```bash
# Install mahara
sudo pacman -S mahara

# Enable and start service
sudo systemctl enable --now mahara

# Verify installation
mahara --version
```

### Alpine Linux

```bash
# Install mahara
apk add --no-cache mahara

# Enable and start service
rc-update add mahara default
rc-service mahara start

# Verify installation
mahara --version
```

### openSUSE/SLES

```bash
# Install mahara
sudo zypper install -y mahara

# Enable and start service
sudo systemctl enable --now mahara

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
mahara --version
```

### macOS

```bash
# Using Homebrew
brew install mahara

# Start service
brew services start mahara

# Verify installation
mahara --version
```

### FreeBSD

```bash
# Using pkg
pkg install mahara

# Enable in rc.conf
echo 'mahara_enable="YES"' >> /etc/rc.conf

# Start service
service mahara start

# Verify installation
mahara --version
```

### Windows

```bash
# Using Chocolatey
choco install mahara

# Or using Scoop
scoop install mahara

# Verify installation
mahara --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/mahara

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
mahara --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable mahara

# Start service
sudo systemctl start mahara

# Stop service
sudo systemctl stop mahara

# Restart service
sudo systemctl restart mahara

# Check status
sudo systemctl status mahara

# View logs
sudo journalctl -u mahara -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add mahara default

# Start service
rc-service mahara start

# Stop service
rc-service mahara stop

# Restart service
rc-service mahara restart

# Check status
rc-service mahara status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'mahara_enable="YES"' >> /etc/rc.conf

# Start service
service mahara start

# Stop service
service mahara stop

# Restart service
service mahara restart

# Check status
service mahara status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mahara
brew services stop mahara
brew services restart mahara

# Check status
brew services list | grep mahara
```

### Windows Service Manager

```powershell
# Start service
net start mahara

# Stop service
net stop mahara

# Using PowerShell
Start-Service mahara
Stop-Service mahara
Restart-Service mahara

# Check status
Get-Service mahara
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream mahara_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name mahara.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name mahara.example.com;

    ssl_certificate /etc/ssl/certs/mahara.example.com.crt;
    ssl_certificate_key /etc/ssl/private/mahara.example.com.key;

    location / {
        proxy_pass http://mahara_backend;
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
    ServerName mahara.example.com
    Redirect permanent / https://mahara.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName mahara.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mahara.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/mahara.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend mahara_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/mahara.pem
    redirect scheme https if !{ ssl_fc }
    default_backend mahara_backend

backend mahara_backend
    balance roundrobin
    server mahara1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R mahara:mahara /etc/mahara
sudo chmod 750 /etc/mahara

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status mahara

# View logs
sudo journalctl -u mahara -f

# Monitor resource usage
top -p $(pgrep mahara)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/mahara"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/mahara-backup-$DATE.tar.gz" /etc/mahara /var/lib/mahara

echo "Backup completed: $BACKUP_DIR/mahara-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop mahara

# Restore from backup
tar -xzf /backup/mahara/mahara-backup-*.tar.gz -C /

# Start service
sudo systemctl start mahara
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u mahara -n 100
sudo tail -f /var/log/mahara/mahara.log

# Check configuration
mahara --version

# Check permissions
ls -la /etc/mahara
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep mahara)

# Check disk I/O
iotop -p $(pgrep mahara)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  mahara:
    image: mahara:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/mahara
      - ./data:/var/lib/mahara
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update mahara

# Debian/Ubuntu
sudo apt update && sudo apt upgrade mahara

# Arch Linux
sudo pacman -Syu mahara

# Alpine Linux
apk update && apk upgrade mahara

# openSUSE
sudo zypper update mahara

# FreeBSD
pkg update && pkg upgrade mahara

# Always backup before updates
tar -czf /backup/mahara-pre-update-$(date +%Y%m%d).tar.gz /etc/mahara

# Restart after updates
sudo systemctl restart mahara
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/mahara

# Clean old logs
find /var/log/mahara -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/mahara
```

## Additional Resources

- Official Documentation: https://docs.mahara.org/
- GitHub Repository: https://github.com/mahara/mahara
- Community Forum: https://forum.mahara.org/
- Best Practices Guide: https://docs.mahara.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
