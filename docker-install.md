# Docker Installation Guide

This guide provides step-by-step instructions for installing Docker on various Linux distributions.

## Table of Contents
- [Red Hat Enterprise Linux (RHEL) / CentOS / Rocky Linux](#red-hat-enterprise-linux-rhel--centos--rocky-linux)
- [Ubuntu / Debian](#ubuntu--debian)
- [Amazon Linux 2](#amazon-linux-2)
- [Post-Installation Steps](#post-installation-steps)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

## Red Hat Enterprise Linux (RHEL) / CentOS / Rocky Linux

### Prerequisites
- Root or sudo access
- 64-bit system
- Kernel version 3.10 or higher

### Installation Steps

1. **Switch to root user:**
   ```bash
   sudo su -
   ```

2. **Install required packages:**
   ```bash
   sudo dnf -y install dnf-plugins-core
   ```

3. **Add Docker repository:**
   ```bash
   sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
   ```

4. **Install Docker components:**
   ```bash
   sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
   ```

5. **Start Docker service:**
   ```bash
   sudo systemctl start docker
   ```

6. **Enable Docker to start on boot:**
   ```bash
   sudo systemctl enable docker
   ```

## Ubuntu / Debian

### Prerequisites
- Root or sudo access
- 64-bit system
- Ubuntu 20.04 LTS or newer / Debian 10 or newer

### Installation Steps

1. **Update package index:**
   ```bash
   sudo apt-get update
   ```

2. **Install prerequisites:**
   ```bash
   sudo apt-get install ca-certificates curl gnupg lsb-release
   ```

3. **Add Docker's official GPG key:**
   ```bash
   sudo mkdir -p /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   ```

4. **Set up the repository:**
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **Update package index again:**
   ```bash
   sudo apt-get update
   ```

6. **Install Docker Engine:**
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

7. **Start and enable Docker:**
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

## Amazon Linux 2

### Installation Steps

1. **Update system packages:**
   ```bash
   sudo yum update -y
   ```

2. **Install Docker:**
   ```bash
   sudo yum install docker -y
   ```

3. **Start Docker service:**
   ```bash
   sudo systemctl start docker
   ```

4. **Enable Docker to start on boot:**
   ```bash
   sudo systemctl enable docker
   ```

## Post-Installation Steps

### Add User to Docker Group

To run Docker commands without sudo, add your user to the docker group:

```bash
sudo usermod -aG docker $USER
```

**Note:** You need to log out and log back in for this change to take effect, or run:
```bash
newgrp docker
```

### Configure Docker to Start on Boot

If not already done during installation:
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## Verification

### Check Docker Version
```bash
docker --version
```

### Check Docker Service Status
```bash
sudo systemctl status docker
```

### Run Test Container
```bash
docker run hello-world
```

### Check Docker Info
```bash
docker info
```

## Troubleshooting

### Common Issues

1. **Permission denied error:**
   - Make sure your user is in the docker group
   - Restart your session after adding user to group

2. **Docker daemon not running:**
   ```bash
   sudo systemctl start docker
   ```

3. **SELinux issues (RHEL/CentOS):**
   ```bash
   sudo setsebool -P container_manage_cgroup on
   ```

4. **Firewall issues:**
   ```bash
   sudo firewall-cmd --permanent --zone=trusted --add-interface=docker0
   sudo firewall-cmd --reload
   ```

### Useful Commands

- **Start Docker:** `sudo systemctl start docker`
- **Stop Docker:** `sudo systemctl stop docker`
- **Restart Docker:** `sudo systemctl restart docker`
- **View Docker logs:** `sudo journalctl -u docker.service`
- **Remove Docker:** Follow uninstallation guide for your distribution

## Security Considerations

1. **Keep Docker updated:** Regularly update Docker to the latest version
2. **Use non-root containers:** Run containers as non-root users when possible
3. **Limit container resources:** Use resource constraints to prevent resource exhaustion
4. **Scan images:** Use tools like `docker scan` to check for vulnerabilities
5. **Use official images:** Prefer official images from Docker Hub

## Additional Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)

---

**Note:** Always refer to the [official Docker documentation](https://docs.docker.com/engine/install/) for the most up-to-date installation instructions and system requirements.