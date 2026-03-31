# Prometheus and Grafana Installation

This document contains the step-by-step instructions for installing and configuring Prometheus and Node Exporter across multiple servers and Grafana.

## Table of Contents
- [Installing Prometheus on Server-1](#installing-prometheus-on-server-1)
- [Installing Prometheus Node Exporter on Server-2](#installing-prometheus-node-exporter-on-server-2)
- [Prometheus Configuration](#prometheus-configuration)
- [Installing Grafana on Server-1](#installing-grafana-on-server-1)

---

## Installing Prometheus on Server-1

### Download and Extract Prometheus

```bash
cd /opt/

wget https://github.com/prometheus/prometheus/releases/download/v3.10.0/prometheus-3.10.0.linux-amd64.tar.gz

tar -xf prometheus-3.10.0.linux-amd64.tar.gz

mv prometheus-3.10.0.linux-amd64 prometheus
```

### Create User and Set Permissions

```bash
sudo mkdir -p /opt/prometheus/data
sudo useradd --no-create-home --shell /sbin/nologin prometheus
sudo chown -R prometheus:prometheus /opt/prometheus
```

### Create Systemd Service

Create the service file:

```bash
vim /etc/systemd/system/prometheus.service
```

Add the following configuration:

```ini
[Unit]
Description=Prometheus Server

[Service]
User=prometheus
Group=prometheus
ExecStart=/opt/prometheus/prometheus \
  --config.file=/opt/prometheus/prometheus.yml \
  --storage.tsdb.path=/opt/prometheus/data \
  --storage.tsdb.retention.time=30d
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Start and Enable Prometheus Service

```bash
systemctl daemon-reload
systemctl enable prometheus
systemctl start prometheus
systemctl status prometheus
```

### Verify Installation

```bash
netstat -lntp  # Shows Port-9090
```

Prometheus should now be running on port 9090.

---

## Installing Prometheus Node Exporter on Server-2

### Download and Extract Node Exporter

```bash
cd /opt/

wget https://github.com/prometheus/node_exporter/releases/download/v1.10.2/node_exporter-1.10.2.linux-amd64.tar.gz

tar -xf node_exporter-1.10.2.linux-amd64.tar.gz

mv node_exporter-1.10.2.linux-amd64 node_exporter
```

### Create User and Set Permissions

```bash
sudo useradd --no-create-home --shell /sbin/nologin node_exporter
sudo chown -R node_exporter:node_exporter /opt/node_exporter
```

### Create Systemd Service

Create the service file:

```bash
vim /etc/systemd/system/node_exporter.service
```

Add the following configuration:

```ini
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
Group=node_exporter
ExecStart=/opt/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
```

### Start and Enable Node Exporter Service

```bash
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
systemctl status node_exporter
```

### Verify Installation

```bash
netstat -lntp  # Shows Port-9100
```

### Configure Firewall

```bash
sudo firewall-cmd --add-port=9100/tcp --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

Node Exporter should now be running on port 9100.

---

## Prometheus Configuration

### Configure prometheus.yml on Server-1

Edit the Prometheus configuration file:

```bash
vim /opt/prometheus/prometheus.yml
```

Add the following configuration:

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
       # The label name is added as a label `label_name=<label_value>` to any timeseries scraped from this config.
        labels:
          app: "prometheus"

  - job_name: "node_exporter"
    static_configs:
      - targets: ["<Server-2-IP>:9100"]
        labels:
          instance: "prod-node-1"
          env: "production"
```

**Note:** Replace `<Server-2-IP>` with the actual IP address of Server-2.

### Restart Prometheus Service

```bash
systemctl restart prometheus
systemctl status prometheus
```

---

---

## Installing Grafana on Server-1

### Import GPG Key

```bash
cd /opt/prometheus/

wget -q -O gpg.key https://rpm.grafana.com/gpg.key
sudo rpm --import gpg.key
```

### Create Grafana Repository Configuration

Create `/etc/yum.repos.d/grafana.repo` with the following content:

```bash
sudo tee /etc/yum.repos.d/grafana.repo > /dev/null <<EOF
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF
```

### Install Grafana OSS

```bash
sudo dnf install grafana
```

### Start and Enable Grafana Service

```bash
systemctl daemon-reload
systemctl enable grafana-server
systemctl start grafana-server
systemctl status grafana-server
```

### Verify Installation

```bash
netstat -lntp  # Shows Port-3000
```

Grafana should now be running on port 3000.

---

## Access Points

- **Prometheus Web UI:** `http://<Server-1-IP>:9090`
- **Grafana Web UI:** `http://<Server-1-IP>:3000`
- **Node Exporter Metrics:** `http://<Server-2-IP>:9100/metrics`

## Default Grafana Credentials

- **Username:** admin
- **Password:** admin

(Change these credentials immediately after first login)
