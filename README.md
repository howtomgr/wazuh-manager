# wazuh Installation Guide

wazuh is a free and open-source security monitoring platform. Fork of OSSEC, Wazuh provides threat detection, integrity monitoring, and compliance, serving as an alternative to Splunk Enterprise Security

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
  - RAM: 2GB minimum (8GB+ recommended)
  - Storage: 10GB for data
  - Network: HTTPS for API
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 55000 (default wazuh port)
  - Ports 1514-1516 for agents
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

# Install wazuh
sudo dnf install -y wazuh-manager

# Enable and start service
sudo systemctl enable --now wazuh-manager

# Configure firewall
sudo firewall-cmd --permanent --add-port=55000/tcp
sudo firewall-cmd --reload

# Verify installation
wazuh-control info
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install wazuh
sudo apt install -y wazuh-manager

# Enable and start service
sudo systemctl enable --now wazuh-manager

# Configure firewall
sudo ufw allow 55000

# Verify installation
wazuh-control info
```

### Arch Linux

```bash
# Install wazuh
sudo pacman -S wazuh-manager

# Enable and start service
sudo systemctl enable --now wazuh-manager

# Verify installation
wazuh-control info
```

### Alpine Linux

```bash
# Install wazuh
apk add --no-cache wazuh-manager

# Enable and start service
rc-update add wazuh-manager default
rc-service wazuh-manager start

# Verify installation
wazuh-control info
```

### openSUSE/SLES

```bash
# Install wazuh
sudo zypper install -y wazuh-manager

# Enable and start service
sudo systemctl enable --now wazuh-manager

# Configure firewall
sudo firewall-cmd --permanent --add-port=55000/tcp
sudo firewall-cmd --reload

# Verify installation
wazuh-control info
```

### macOS

```bash
# Using Homebrew
brew install wazuh-manager

# Start service
brew services start wazuh-manager

# Verify installation
wazuh-control info
```

### FreeBSD

```bash
# Using pkg
pkg install wazuh-manager

# Enable in rc.conf
echo 'wazuh-manager_enable="YES"' >> /etc/rc.conf

# Start service
service wazuh-manager start

# Verify installation
wazuh-control info
```

### Windows

```bash
# Using Chocolatey
choco install wazuh-manager

# Or using Scoop
scoop install wazuh-manager

# Verify installation
wazuh-control info
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/wazuh-manager

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
wazuh-control info
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable wazuh-manager

# Start service
sudo systemctl start wazuh-manager

# Stop service
sudo systemctl stop wazuh-manager

# Restart service
sudo systemctl restart wazuh-manager

# Check status
sudo systemctl status wazuh-manager

# View logs
sudo journalctl -u wazuh-manager -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add wazuh-manager default

# Start service
rc-service wazuh-manager start

# Stop service
rc-service wazuh-manager stop

# Restart service
rc-service wazuh-manager restart

# Check status
rc-service wazuh-manager status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'wazuh-manager_enable="YES"' >> /etc/rc.conf

# Start service
service wazuh-manager start

# Stop service
service wazuh-manager stop

# Restart service
service wazuh-manager restart

# Check status
service wazuh-manager status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start wazuh-manager
brew services stop wazuh-manager
brew services restart wazuh-manager

# Check status
brew services list | grep wazuh-manager
```

### Windows Service Manager

```powershell
# Start service
net start wazuh-manager

# Stop service
net stop wazuh-manager

# Using PowerShell
Start-Service wazuh-manager
Stop-Service wazuh-manager
Restart-Service wazuh-manager

# Check status
Get-Service wazuh-manager
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream wazuh-manager_backend {
    server 127.0.0.1:55000;
}

server {
    listen 80;
    server_name wazuh-manager.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name wazuh-manager.example.com;

    ssl_certificate /etc/ssl/certs/wazuh-manager.example.com.crt;
    ssl_certificate_key /etc/ssl/private/wazuh-manager.example.com.key;

    location / {
        proxy_pass http://wazuh-manager_backend;
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
    ServerName wazuh-manager.example.com
    Redirect permanent / https://wazuh-manager.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName wazuh-manager.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/wazuh-manager.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/wazuh-manager.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:55000/
    ProxyPassReverse / http://127.0.0.1:55000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend wazuh-manager_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/wazuh-manager.pem
    redirect scheme https if !{ ssl_fc }
    default_backend wazuh-manager_backend

backend wazuh-manager_backend
    balance roundrobin
    server wazuh-manager1 127.0.0.1:55000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R wazuh-manager:wazuh-manager /etc/wazuh-manager
sudo chmod 750 /etc/wazuh-manager

# Configure firewall
sudo firewall-cmd --permanent --add-port=55000/tcp
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
sudo systemctl status wazuh-manager

# View logs
sudo journalctl -u wazuh-manager -f

# Monitor resource usage
top -p $(pgrep wazuh-manager)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/wazuh-manager"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/wazuh-manager-backup-$DATE.tar.gz" /etc/wazuh-manager /var/lib/wazuh-manager

echo "Backup completed: $BACKUP_DIR/wazuh-manager-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop wazuh-manager

# Restore from backup
tar -xzf /backup/wazuh-manager/wazuh-manager-backup-*.tar.gz -C /

# Start service
sudo systemctl start wazuh-manager
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u wazuh-manager -n 100
sudo tail -f /var/log/wazuh-manager/wazuh-manager.log

# Check configuration
wazuh-control info

# Check permissions
ls -la /etc/wazuh-manager
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 55000

# Test connectivity
telnet localhost 55000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep wazuh-manager)

# Check disk I/O
iotop -p $(pgrep wazuh-manager)

# Check connections
ss -an | grep 55000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  wazuh-manager:
    image: wazuh-manager:latest
    ports:
      - "55000:55000"
    volumes:
      - ./config:/etc/wazuh-manager
      - ./data:/var/lib/wazuh-manager
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update wazuh-manager

# Debian/Ubuntu
sudo apt update && sudo apt upgrade wazuh-manager

# Arch Linux
sudo pacman -Syu wazuh-manager

# Alpine Linux
apk update && apk upgrade wazuh-manager

# openSUSE
sudo zypper update wazuh-manager

# FreeBSD
pkg update && pkg upgrade wazuh-manager

# Always backup before updates
tar -czf /backup/wazuh-manager-pre-update-$(date +%Y%m%d).tar.gz /etc/wazuh-manager

# Restart after updates
sudo systemctl restart wazuh-manager
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/wazuh-manager

# Clean old logs
find /var/log/wazuh-manager -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/wazuh-manager
```

## Additional Resources

- Official Documentation: https://docs.wazuh-manager.org/
- GitHub Repository: https://github.com/wazuh-manager/wazuh-manager
- Community Forum: https://forum.wazuh-manager.org/
- Best Practices Guide: https://docs.wazuh-manager.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
