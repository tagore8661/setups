# TERRAFORM INSTALLATIONS

**If you're looking for the latest Terraform version, visiting the official documentation :**
```
https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli#install-terraform
```

* Steps to install Terraform on a Red Hat-based system -

Switch to the root user:
```
sudo su -
```

Install yum-config-manager to manage your repositories:
```
sudo yum install -y yum-utils
```

Use yum-config-manager to add the official HashiCorp Linux repository:
```
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

Install Terraform from the new repository:
```
sudo yum -y install terraform
```

Verify the installation
```
terraform -help
```