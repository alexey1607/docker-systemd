### Контейнер с systemd

#### Образа
```
debian12: alexey1607/systemd:debian12
debian13: alexey1607/systemd:debian13
```

#### Запуск
```
docker run -d \
  --privileged \
  --cgroupns=host \
  -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
  --name debian-systemd \
  alexey1607/systemd:debian12
```
