# Bare Metal Setup - Installation Guide

## Perform Base OS Setup and Package Installation

### Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Core System Packages

```bash
sudo apt install -y \
  curl wget git htop nano vim unzip ca-certificates gnupg \
  lsb-release net-tools sysstat iotop jq
```
```bash
sudo apt install -y   ipmitool smartmontools lm-sensors nvme-cli
```
### Configure Automatic Security Updates

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```
### Install docker and docker compose

```
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version

```
## Node Exporter Installation

### Download Node Exporter for AMD64

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/\
download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter \
  /usr/local/bin/
```

### Download Node Exporter for ARM64

```bash
wget https://github.com/prometheus/node_exporter/releases/\
download/v1.7.0/node_exporter-1.7.0.linux-arm64.tar.gz

# Extract the tarball
tar xvf node_exporter-1.7.0.linux-arm64.tar.gz

# Move the binary to /usr/local/bin so it's in your PATH
sudo mv node_exporter-1.7.0.linux-arm64/node_exporter \
  /usr/local/bin/

# Optional: check installation
node_exporter --version
```

### Configure Node Exporter as a Systemd Service

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
sudo systemctl status node_exporter

```

### Verify Node Exporter is Running

```bash
curl http://localhost:9100/metrics
```

## Other Exporters Setup

### Clone the GitHub Repository

```bash
git clone https://github.com/opshealth/gpu-monitoring.git
```

### Build the Unified Simulator

```bash
cd gpu-monitoring
docker-compose build unified-simulator
```

### Start the Unified Simulator

```bash
docker-compose up -d unified-simulator
```
```
refer: https://github.com/opshealth/gpu-monitoring/blob/main/unified-simulator/README.md
```
### Check Logs

```bash
docker logs unified_simulator
```
### ensure that you have enable all the ports 

