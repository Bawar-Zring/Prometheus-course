# Promtail — example configuration with per-block explanations

This document shows a complete Promtail configuration and explains each block briefly after the full config. Use the config as a starting point and update the placeholders (Loki URL, paths, selectors) for your environment.

## Full configuration (ready-to-use example)

```
server:
  http_listen_port: 9080
  grpc_listen_port: 0
```
### server
Purpose: configures the internal HTTP/gRPC endpoints Promtail exposes.
In the example:
- http_listen_port: 9080 — Promtail will serve metrics and a health endpoint on port 9080.
- grpc_listen_port: 0 — disables gRPC listener (common if not needed).

---

```
clients:
  - url: http://<LOKI-SERVER>:3100/loki/api/v1/push
    batchsize: 5120
    batchwait: 1s
    backoff_config:
      min_period: 500ms
      max_period: 5m
      max_retries: 10
```
### clients
Purpose: where Promtail should push logs (Loki instances).
In the example:
- url points to your Loki push API endpoint.
- batchsize and batchwait control batching to optimize throughput.
- backoff_config controls retry behavior on push failures.
- Tip: enable basic_auth or TLS if your Loki is protected.

---

```
positions:
  filename: /tmp/positions.yaml
```
### positions
Purpose: file path where Promtail stores the last read offsets ("positions") for each tailed source.

```
scrape_configs:
  - job_name: docker-logs
    # Discover running Docker containers (Unix socket)
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s

    # Optional pipeline stages to parse and filter logs
    pipeline_stages:
      # Parse Docker JSON log format into structured fields
      - docker: {}

      # Example: add or rename labels using regex (useful to normalize container names)
      - regex:
          expression: '^(?P<ts>[^ ]+) (?P<stream>stdout|stderr) (?P<message>.*)$'
      - labels:
          - stream

      # Match stage to conditionally drop logs
      - match:
          selector: '{container="nginx"}'
          stages:
            - drop:
                expression: '.*GET /health.*'

    # Relabels translate auto-discovered metadata into labels sent to Loki
    relabel_configs:
      # Convert Docker container name (which often starts with '/') into `container` label
      - source_labels: [__meta_docker_container_name]
        regex: '^/(.+)'
        replacement: '$1'
        target_label: container

    # Optional: use static labels (applied to every log of this scrape job)
    static_configs:
      - targets: ['localhost']
        labels:
          job: docker-logs
          environment: staging
```
### scrape_configs
Purpose: defines jobs to discover log sources and how to process them.
The job_name groups sources. Each scrape job can use multiple discovery methods (docker_sd_configs, static_configs, file, etc.).
Subsections used in the example:

- docker_sd_configs
    - Uses Docker socket to discover container metadata.
    - host: unix:///var/run/docker.sock is the typical socket for Linux hosts. On Windows Docker Desktop you may need to use a TCP endpoint or run Promtail as a container with socket mount.
      
- pipeline_stages
    - A sequence of transformations applied to each log entry before pushing to Loki.
    - docker: {} parses Docker JSON logs into structured fields (timestamp, stream, message).
    - regex and labels extract fields and convert them into labels.
    - match allows conditional stages; in this example it drops health-check requests coming from the nginx container.
      
- relabel_configs
    - Maps Prometheus-style metadata from discovery into labels used in Loki.
    - Useful to sanitize values (e.g., strip the leading slash from Docker container names).
      
- static_configs
    - Attach static labels to every log in this job (e.g., job, environment).
