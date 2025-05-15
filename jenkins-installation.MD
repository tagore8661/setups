# JENKINS INSTALLATIONS

**If you're looking for the latest Jenkins version, visiting the official documentation :**
```
https://www.jenkins.io/doc/book/installing/linux/
```

* Steps to install Jenkins on a Red Hat-based system -

Switch to the root user:
```
sudo su -
```

Download the Jenkins repo file:
```
sudo curl -o /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

Import the Jenkins GPG key:
```
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

Upgrade system packages:
```
sudo yum upgrade
```

Install dependencies:
```
sudo yum install fontconfig java-21-openjdk
```

Install Jenkins:
```
sudo yum install jenkins
```

Reload system daemons:
```
sudo systemctl daemon-reload
```

You can enable the Jenkins service to start at boot with the command:
```
sudo systemctl enable jenkins
```

You can start the Jenkins service with the command:
```
sudo systemctl start jenkins
```

You can check the status of the Jenkins service using the command:
```
sudo systemctl status jenkins
```