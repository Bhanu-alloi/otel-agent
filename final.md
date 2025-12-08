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
### List of simulators
```
gpu_simulator
infiniband_simulator
ipmi_simulator
kernel_log_simulator
kubernetes_simulator
log_simulator
loki_logger
netq_simulator
network_simulator
process_simulator
rafay_inventory_simulator
raid_simulator
security_simulator
smart_simulator
vm_simulator
weka_simulator
```

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
docker run -d   --name unified_simulator   --add-host=host.docker.internal:host-gateway   -e OTEL_EXPORTER_OTLP_ENDPOINT=host.docker.internal:4317     -p 8010:8010   -p 9118:9118   -p 9256:9256   -p 9290:9290   -p 9400:9400   -p 9537:9537   -p 9600:9600   -p 9627:9627   -p 9633:9633   -p 9640:9640   -p 9650:9650   -p 9700:9700   -p 9800:9800   gpu-monitoring-unified-simulator
```
```
refer: https://github.com/opshealth/gpu-monitoring/blob/main/unified-simulator/README.md
```
### Check Logs

```bash
docker logs unified_simulator
```
### ensure that you have enable all the ports 


## OpenTelemetry Collector Agent - Installation and Setup
## method-1

## 1. Download Binary

### For AMD64 (x86_64) Architecture:
```bash
wget https://github.com/open-telemetry/opentelemetry-collector-releases/\
releases/download/v0.136.0/otelcol-contrib_0.136.0_linux_amd64.tar.gz
```

### For ARM64 (aarch64) Architecture:
```bash
wget https://github.com/open-telemetry/opentelemetry-collector-releases/\
releases/download/v0.136.0/otelcol-contrib_0.136.0_linux_arm64.tar.gz
```

## 2. Extract and Install

### For AMD64:
```bash
tar -xvzf otelcol-contrib_0.136.0_linux_amd64.tar.gz
sudo mv otelcol-contrib /usr/local/bin/otelcol
otelcol --version
```

### For ARM64:
```bash
tar -xvzf otelcol-contrib_0.136.0_linux_arm64.tar.gz
sudo mv otelcol-contrib /usr/local/bin/otelcol
otelcol --version
```

## 3. Create Config Directory

```bash
sudo mkdir -p /etc/otel-collector
```

## 3.1. OpenTelemetry Collector Agent Configuration

```bash
sudo nano /etc/otel-collector/otel-agent-config.yaml
```
```
refer: https://github.com/opshealth/gpu-monitoring/blob/main/otel-collector-config.yaml
```
```
below is only some part of config of otel collector agent
```
```yaml
receivers:
  # Prometheus receivers for different device categories
  prometheus/baremetal-1:
    config:
      scrape_configs:
        - job_name: 'baremetal-server-1'
          scrape_interval: 15s
          static_configs:
            - targets: 
                - '<baremetal-server-ip>:9100'  # replace baremetal-server-ip with the public ip
                - '<baremetal-server-ip>:9290'
                - '<baremetal-server-ip>:9633'

  prometheus/ib-switch-1:
    config:
      scrape_configs:
        - job_name: 'infiniband-switch-1'
          scrape_interval: 15s
          static_configs:
            - targets: 
                - '<ib-switch-ip>:9627'

  prometheus/netq-switch-1:
    config:
      scrape_configs:
        - job_name: 'netq-switch-1'
          static_configs:
            - targets: 
                - '<netq-switch-ip>:9650'

  prometheus/weka-storage-1:
    config:
      scrape_configs:
        - job_name: 'weka-storage-1'
          static_configs:
            - targets: 
                - '<weka-storage-ip>:9640'

  prometheus/gpu-server-1:
    config:
      scrape_configs:
        - job_name: 'gpu-server-1'
          static_configs:
            - targets: 
                - '<gpu-server-ip>:9100'
                - '<gpu-server-ip>:9400'

processors:
  batch: {}
  
  # Example metric transform processor to add custom labels
  metricstransform/baremetal-1:
    transforms:
      - include: ".*"
        match_type: regexp
        action: update
        operations:
          - action: add_label
            new_label: device_id
            new_value: "baremetal-001"
          - action: add_label
            new_label: device_type
            new_value: "Baremetal"

  metricstransform/ib-switch-1:
    transforms:
      - include: ".*"
        match_type: regexp
        action: update
        operations:
          - action: add_label
            new_label: device_id
            new_value: "ib-switch-001"
          - action: add_label
            new_label: device_type
            new_value: "Switch"

exporters:
  otlphttp:
    endpoint: "http://<your-otlp-endpoint>:4318"    #replace withe loadbalancer of otel collector

service:
  pipelines:
    metrics/baremetal-1:
      receivers: [prometheus/baremetal-1]
      processors: [metricstransform/baremetal-1, batch]
      exporters: [otlphttp]
    
    metrics/ib-switch-1:
      receivers: [prometheus/ib-switch-1]
      processors: [metricstransform/ib-switch-1, batch]
      exporters: [otlphttp]
```

## 3.2. Systemd Service Configuration

```bash
sudo nano /etc/systemd/system/otel-collector-agent.service
```

```ini
[Unit]
Description=OpenTelemetry Collector Agent
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/local/bin/otelcol --config /etc/otel-collector/otel-agent-config.yaml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable otel-collector-agent.service
sudo systemctl start otel-collector-agent.service
sudo systemctl status otel-collector-agent.service
```
## method-2
### otel agent setup using docker
## 1. Create Config Directory

```bash
sudo mkdir -p /etc/otel-collector
```

## 2. OpenTelemetry Collector Agent Configuration

```bash
sudo nano /etc/otel-collector/otel-agent-config.yaml
```
## 3. Docker run command
```
docker run -d --network host --name otel-collector \
  -v /etc/otel-collector/otel-agent-config.yaml:/etc/otel-collector/otel-agent-config.yaml \
  otel/opentelemetry-collector-contrib:latest \
  --config /etc/otel-collector/otel-agent-config.yaml
```
## 4. check logs
```
docker ps
docker logs -f otel-collector
```
