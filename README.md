# Alloy - Docker Log Collection to Loki

This project uses [Grafana Alloy](https://grafana.com/docs/alloy/latest/) to collect logs from Docker containers and forward them to Loki for centralized logging.

## Overview

Alloy is Grafana's configuration language for observability. This setup:
- **Discovers** Docker containers running on the host
- **Collects** logs from all discovered containers
- **Relabels** logs with metadata from Docker labels (app, service_name, environment)
- **Forwards** logs to a Loki instance for storage and querying

## Prerequisites

- Docker and Docker Compose installed
- A running Loki instance (default: `http://localhost:3100`)
- Docker socket access (for container discovery)

## Project Structure

```
.
├── config.alloy          # Alloy configuration file
├── docker-compose.yml    # Docker Compose setup for Alloy
└── README.md            # This file
```

## Configuration

### `config.alloy`

The Alloy configuration defines:

1. **Docker Discovery** (`discovery.docker`): Discovers containers every 5 seconds
2. **Relabeling Rules** (`discovery.relabel`): Extracts metadata from Docker labels:
   - `app` → from `__meta_docker_container_label_app`
   - `service_name` → from `__meta_docker_container_label_service_name`
   - `environment` → from `__meta_docker_container_label_environment`
3. **Log Collection** (`loki.source.docker`): Collects logs from discovered containers
4. **Log Forwarding** (`loki.write`): Sends logs to Loki at `http://localhost:3100`

### Customization

To modify the Loki endpoint, edit the `url` in `loki.write.default.endpoint`:

```alloy
loki.write "default" {
  endpoint {
    url = "http://your-loki-host:3100/loki/api/v1/push"
  }
  external_labels = {
    host = "your-hostname",
  }
}
```

## Usage

### Starting Alloy

1. Ensure Loki is running and accessible at `http://localhost:3100`
2. Start Alloy using Docker Compose:

```bash
docker compose up -d
```

### Viewing Logs

To view Alloy container logs:

```bash
docker compose logs -f alloy
```

### Stopping Alloy

```bash
docker compose down
```

## Docker Labels

To take advantage of the relabeling rules, add the following labels to your containers:

```yaml
labels:
  app: "my-application"
  service_name: "my-service"
  environment: "production"
```

These labels will be automatically extracted and added as labels to your log entries in Loki.

## Verification

After starting Alloy, you can verify it's working by:

1. Checking that the Alloy container is running:
   ```bash
   docker ps | grep alloy
   ```

2. Querying Loki to see if logs are being received:
   ```bash
   curl -G -s "http://localhost:3100/loki/api/v1/label/__name__/values"
   ```

3. Viewing Alloy metrics (if enabled):
   ```bash
   curl http://localhost:12345/metrics
   ```

## Troubleshooting

### Alloy can't connect to Loki

- Verify Loki is running: `docker ps | grep loki`
- Check the Loki URL in `config.alloy` matches your Loki instance
- Ensure network connectivity between Alloy and Loki containers

### No logs being collected

- Verify Docker socket is mounted correctly: `/var/run/docker.sock`
- Check that containers are running: `docker ps`
- Review Alloy logs: `docker compose logs alloy`

### Permission issues

- Ensure the Docker socket has proper permissions
- On Linux, you may need to add the user to the `docker` group

## Resources

- [Grafana Alloy Documentation](https://grafana.com/docs/alloy/latest/)
- [Alloy Docker Source](https://grafana.com/docs/alloy/latest/reference/components/loki.source.docker/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)

## License

This project is provided as-is for demonstration purposes.

