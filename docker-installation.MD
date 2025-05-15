# DOCKER INSTALLATIONS

**If you're looking for the latest Docker version, visiting the official documentation :**
```
https://docs.docker.com/engine/install/rhel/
```

* Steps to install Docker on a Red Hat-based system -

Switch to the root user:
```
sudo su -
```

Install DNF plugins:
```
sudo dnf -y install dnf-plugins-core
```

Add the Docker repository:
```
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

Install Docker components:
```
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Start the Docker service:
```
sudo systemctl start docker
```

Enable Docker on startup:
```
sudo systemctl enable docker
```

Grant ec2-user Docker privileges (Allows ec2-user to execute Docker commands without sudo):
```
sudo usermod -aG docker ec2-user
```