## Run container with no privilege escalation

```
docker run -it --rm --security-opt=no-new-privileges busybox sh
```

## Run nginx with minimal capabilities
```
docker run -it --rm --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx:latest
```

## Run container in complete network isolation
```
docker run -it --rm --network=none debian
```

## Run container on specific CPU cores
```
docker run -it --rm --cpuset-cpus="2,3" ubuntu
```

## Run container with custom timezone
```
docker run -it --rm -e TZ=America/New_York alpine date
```