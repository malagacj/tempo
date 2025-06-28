# Tempo Distributed Tracing Setup

This Docker Compose configuration sets up Grafana Tempo for distributed tracing storage and visualization.

### Services Overview

#### tempo-init  
**Purpose**: Initialization container that sets proper permissions for Tempo data directory.  
**Image**: busybox  
**Function**: Ensures the `/tmp/tempo` directory has correct write permissions (777).

#### tempo  
**Purpose**: Main Tempo service for trace storage and retrieval.  
**Image**: grafana/tempo:latest  
**Configuration**: Uses external `tempo.yaml` config file.  
**Data Storage**: Persistent volume for trace data.

### Prerequisites

You'll need Docker and Docker Compose installed, a `tempo.yaml` configuration file in the same directory, and the external network `super_network` must exist.

### Configuration Files Required

#### tempo.yaml
You need to create a `tempo.yaml` configuration file. Basic example:

```yaml
auth_enabled: false

server:
  http_listen_port: 3200
  grpc_listen_port: 9095

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318

ingester:
  trace_idle_period: 10s
  max_block_bytes: 1_000_000
  max_block_duration: 5m

compactor:
  compaction:
    block_retention: 1h

storage:
  trace:
    backend: local
    local:
      path: /tmp/tempo/traces
    wal:
      path: /tmp/tempo/wal
    pool:
      max_workers: 100
      queue_depth: 10000
```

### Network Setup

Create the external network if it doesn't exist:
```bash
docker network create super_network
```

### Usage

1. Ensure `tempo.yaml` exists in the same directory
2. Start the services:
   ```bash
   docker-compose up -d
   ```

### Exposed Ports

**3200**: Tempo HTTP API (for Grafana integration)  
**14268**: Jaeger trace ingestion endpoint

#### Commented Ports
**4317**: OTLP gRPC receiver (disabled - uncomment if needed)  
**4318**: OTLP HTTP receiver (disabled - uncomment if needed)

### Data Persistence

**Volume**: `tempo-data` stores all trace data persistently  
**Mount Point**: `/tmp/tempo` inside containers

### Integration

#### With Grafana
Configure Grafana data source:
- **Type**: Tempo
- **URL**: `http://tempo:3200`

#### Sending Traces
Send traces to:
- **Jaeger format**: `http://localhost:14268/api/traces`
- **OTLP** (if enabled): `http://localhost:4317` (gRPC) or `http://localhost:4318` (HTTP)

### Troubleshooting

1. **Permission Issues**: The `tempo-init` service should resolve permission problems
2. **Network Issues**: Ensure `super_network` exists and is accessible
3. **Config Issues**: Verify `tempo.yaml` syntax and paths

### Cleanup

```bash
docker-compose down -v  # Removes containers and volumes
```

**Note**: This will delete all stored trace data.
