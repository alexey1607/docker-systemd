# docker-systemd

A Docker image for running systemd inside a container. This allows you to run services that depend on systemd initialization within a containerized environment.

## Features

- Based on Ubuntu 22.04
- Systemd fully functional inside the container
- Minimal image with unnecessary systemd services removed
- Ready to use with docker-compose
- Example service included

## Prerequisites

- Docker Engine 20.10 or higher
- Docker Compose (optional, for using docker-compose.yml)

## Quick Start

### Using Docker

Build the image:
```bash
docker build -t systemd-docker .
```

Run the container:
```bash
docker run -d \
  --name systemd-container \
  --privileged \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --tmpfs /tmp \
  --cap-add SYS_ADMIN \
  --security-opt seccomp:unconfined \
  --stop-signal SIGRTMIN+3 \
  -t systemd-docker
```

### Using Docker Compose

Simply run:
```bash
docker-compose up -d
```

## Accessing the Container

Execute commands inside the running container:
```bash
docker exec -it systemd-container bash
```

Check systemd status:
```bash
docker exec -it systemd-container systemctl status
```

## Running Custom Services

### Method 1: Build Time (Recommended)

Create a custom Dockerfile based on this image:
```dockerfile
FROM systemd-docker:latest

# Copy your service file
COPY my-service.service /etc/systemd/system/my-service.service

# Enable the service
RUN systemctl enable my-service.service
```

### Method 2: Runtime

Copy service file to running container:
```bash
docker cp my-service.service systemd-container:/etc/systemd/system/
docker exec -it systemd-container systemctl daemon-reload
docker exec -it systemd-container systemctl enable my-service.service
docker exec -it systemd-container systemctl start my-service.service
```

## Example

An example service is provided in the `examples/` directory. To use it:

```bash
cd examples
docker build -f Dockerfile.example -t systemd-docker-example ..
docker run -d \
  --name systemd-example \
  --privileged \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  --tmpfs /run \
  --tmpfs /run/lock \
  --tmpfs /tmp \
  --cap-add SYS_ADMIN \
  --security-opt seccomp:unconfined \
  --stop-signal SIGRTMIN+3 \
  -t systemd-docker-example
```

View the example service logs:
```bash
docker exec -it systemd-example journalctl -u hello.service -f
```

## Important Notes

### Privileged Mode
The container must run in privileged mode for systemd to function properly. This is required for cgroup management and systemd's init process.

### Volume Mounts
- `/sys/fs/cgroup` must be mounted (read-only is sufficient)
- `/run`, `/run/lock`, and `/tmp` should be tmpfs mounts

### Stopping the Container
Use `SIGRTMIN+3` signal for graceful shutdown:
```bash
docker stop systemd-container
```

### Security Considerations
Running containers in privileged mode has security implications. Only use this in trusted environments or where systemd is absolutely necessary.

## Troubleshooting

### Systemd fails to start
- Ensure the container is running with `--privileged` flag
- Check that `/sys/fs/cgroup` is properly mounted
- Verify tmpfs mounts are configured

### Service won't start
```bash
# Check service status
docker exec -it systemd-container systemctl status my-service

# View service logs
docker exec -it systemd-container journalctl -u my-service -n 50

# Reload systemd
docker exec -it systemd-container systemctl daemon-reload
```

## License

This project is provided as-is for running systemd in Docker containers.