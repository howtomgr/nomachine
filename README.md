# nomachine Installation Guide

nomachine is a free and open-source remote desktop. NoMachine provides fast remote desktop access

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
  - Storage: 2GB for sessions
  - Network: NX protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4000 (default nomachine port)
  - Various services
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

# Install nomachine
sudo dnf install -y nomachine

# Enable and start service
sudo systemctl enable --now nomachine

# Configure firewall
sudo firewall-cmd --permanent --add-port=4000/tcp
sudo firewall-cmd --reload

# Verify installation
nomachine --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nomachine
sudo apt install -y nomachine

# Enable and start service
sudo systemctl enable --now nomachine

# Configure firewall
sudo ufw allow 4000

# Verify installation
nomachine --version
```

### Arch Linux

```bash
# Install nomachine
sudo pacman -S nomachine

# Enable and start service
sudo systemctl enable --now nomachine

# Verify installation
nomachine --version
```

### Alpine Linux

```bash
# Install nomachine
apk add --no-cache nomachine

# Enable and start service
rc-update add nomachine default
rc-service nomachine start

# Verify installation
nomachine --version
```

### openSUSE/SLES

```bash
# Install nomachine
sudo zypper install -y nomachine

# Enable and start service
sudo systemctl enable --now nomachine

# Configure firewall
sudo firewall-cmd --permanent --add-port=4000/tcp
sudo firewall-cmd --reload

# Verify installation
nomachine --version
```

### macOS

```bash
# Using Homebrew
brew install nomachine

# Start service
brew services start nomachine

# Verify installation
nomachine --version
```

### FreeBSD

```bash
# Using pkg
pkg install nomachine

# Enable in rc.conf
echo 'nomachine_enable="YES"' >> /etc/rc.conf

# Start service
service nomachine start

# Verify installation
nomachine --version
```

### Windows

```bash
# Using Chocolatey
choco install nomachine

# Or using Scoop
scoop install nomachine

# Verify installation
nomachine --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nomachine

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nomachine --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nomachine

# Start service
sudo systemctl start nomachine

# Stop service
sudo systemctl stop nomachine

# Restart service
sudo systemctl restart nomachine

# Check status
sudo systemctl status nomachine

# View logs
sudo journalctl -u nomachine -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nomachine default

# Start service
rc-service nomachine start

# Stop service
rc-service nomachine stop

# Restart service
rc-service nomachine restart

# Check status
rc-service nomachine status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nomachine_enable="YES"' >> /etc/rc.conf

# Start service
service nomachine start

# Stop service
service nomachine stop

# Restart service
service nomachine restart

# Check status
service nomachine status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nomachine
brew services stop nomachine
brew services restart nomachine

# Check status
brew services list | grep nomachine
```

### Windows Service Manager

```powershell
# Start service
net start nomachine

# Stop service
net stop nomachine

# Using PowerShell
Start-Service nomachine
Stop-Service nomachine
Restart-Service nomachine

# Check status
Get-Service nomachine
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nomachine_backend {
    server 127.0.0.1:4000;
}

server {
    listen 80;
    server_name nomachine.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nomachine.example.com;

    ssl_certificate /etc/ssl/certs/nomachine.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nomachine.example.com.key;

    location / {
        proxy_pass http://nomachine_backend;
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
    ServerName nomachine.example.com
    Redirect permanent / https://nomachine.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nomachine.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nomachine.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nomachine.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4000/
    ProxyPassReverse / http://127.0.0.1:4000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nomachine_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nomachine.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nomachine_backend

backend nomachine_backend
    balance roundrobin
    server nomachine1 127.0.0.1:4000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nomachine:nomachine /etc/nomachine
sudo chmod 750 /etc/nomachine

# Configure firewall
sudo firewall-cmd --permanent --add-port=4000/tcp
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
sudo systemctl status nomachine

# View logs
sudo journalctl -u nomachine -f

# Monitor resource usage
top -p $(pgrep nomachine)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nomachine"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nomachine-backup-$DATE.tar.gz" /etc/nomachine /var/lib/nomachine

echo "Backup completed: $BACKUP_DIR/nomachine-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nomachine

# Restore from backup
tar -xzf /backup/nomachine/nomachine-backup-*.tar.gz -C /

# Start service
sudo systemctl start nomachine
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nomachine -n 100
sudo tail -f /var/log/nomachine/nomachine.log

# Check configuration
nomachine --version

# Check permissions
ls -la /etc/nomachine
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4000

# Test connectivity
telnet localhost 4000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nomachine)

# Check disk I/O
iotop -p $(pgrep nomachine)

# Check connections
ss -an | grep 4000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nomachine:
    image: nomachine:latest
    ports:
      - "4000:4000"
    volumes:
      - ./config:/etc/nomachine
      - ./data:/var/lib/nomachine
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nomachine

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nomachine

# Arch Linux
sudo pacman -Syu nomachine

# Alpine Linux
apk update && apk upgrade nomachine

# openSUSE
sudo zypper update nomachine

# FreeBSD
pkg update && pkg upgrade nomachine

# Always backup before updates
tar -czf /backup/nomachine-pre-update-$(date +%Y%m%d).tar.gz /etc/nomachine

# Restart after updates
sudo systemctl restart nomachine
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nomachine

# Clean old logs
find /var/log/nomachine -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nomachine
```

## Additional Resources

- Official Documentation: https://docs.nomachine.org/
- GitHub Repository: https://github.com/nomachine/nomachine
- Community Forum: https://forum.nomachine.org/
- Best Practices Guide: https://docs.nomachine.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
