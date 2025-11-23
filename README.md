# docker-systemd

### Запуск контейнера

```
docker run -d \
  --privileged \
  --cgroupns=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
  --name debian-systemd \
  debian12-systemd
```