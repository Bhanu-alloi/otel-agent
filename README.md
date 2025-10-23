# OpenTelemetry Collector Agent - Installation and Setup

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
                - '<baremetal-server-ip>:9100'
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
    endpoint: "http://<your-otlp-endpoint>:4318"

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
