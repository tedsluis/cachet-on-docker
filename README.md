# cachet-on-docker
Cachet and Cachet-monitor running on Docker
  
Explained how to run it on `Raspbian` (boards like `Raspberry Pi` or `Orange Pi`) or any other Linux distros like `Fedora`, `Centos`, `RHEL`, `Debian`, `Ubuntu` and `Mint`.  
  
## Install docker  
One way to install Docker on any kind of Linux distro:  
```
pi@raspberry:~ $ curl -sSL https://get.docker.com | sh
```
More info at [https://docs.docker.com/install/](https://docs.docker.com/install/)  
Note for raspberry: Docker runs nowadays on raspbian. It isn't needed to use the [Hypriot Docker Image for Raspberry Pi](https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/) any more.  
  
## Create data directory for Postgres database  
By default containers use ephemeral storage. To ensure persistent storage, a directory will be mounted into the Postgres container.   
```
pi@raspberry:~ $ sudo mkdir -p /var/lib/progresql/data
```
  
## Clone this git repository  
```
pi@raspberry:~ $ git clone https://github.com/tedsluis/cachet-on-docker.git
pi@raspberry:~ $ cd cachet-on-docker
```  
   
## Init scripts  
Nowadays on most Linux distros `Systemd` manage services, so `Systemd` will be used to launch the containers. Each container has its own `unit file`:  
These `unit files` are configured to run on `arm32v6` (like `Raspberry Pi` or `Orange Pi`). If you want to run these on `X86` or `amd64`, you should change the Docker `image:tag` in these unit files:  

| Unit file                                                                                                   | arm32v6 Docker image                        | X86 / amd64 Docker image                                                                                           |
| :---------------------------------------------------------------------------------------------------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------------------------------- |
| [docker-cachet.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-cachet.service)     | [tedsluis/cachet:2.3.13-nginx-alpine-arm32v6](https://hub.docker.com/r/tedsluis/cachet/) | [cachethq/docker:latest](https://hub.docker.com/r/cachethq/docker/)   |
| [docker-postgres.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-postgres.service) | [arm32v6/postgres:alpine](https://hub.docker.com/r/arm32v6/postgres/)                    | [postgres:9.5](https://hub.docker.com/_/postgres/)                    |
| [cachet-monitor.service](https://github.com/tedsluis/cachet-on-docker/blob/master/cachet-monitor.service)   |                                             |                                                                                                                    |
  
## Start services  
Simply copy these `unit files` from the root of the cloned repository to `/etc/systemd/system/` and do a `systemctl daemon-reload`:  
```
pi@raspberry:~/cachet-on-docker $ sudo cp *.service /etc/systemd/system
pi@raspberry:~/cachet-on-docker $ sudo systemctl daemon-reload
```
Finally you can `enable` these services, so they will always start after a reboot:   
```
pi@raspberry:~/cachet-on-docker $ sudo systemctl enable docker-postgres.service
pi@raspberry:~/cachet-on-docker $ sudo systemctl enable docker-cachet.service
```
  
## Build your own Cachet image for arm32
Unforunately for arm32 users (like Raspberry Pi and Orange Pi) there is no official `Cachet` image available. Users can pull my image `tedsluis/cachet:2.3.13-nginx-alpine-arm32v6` from Dockerhub, or they can build it them self as described below:  
  
### Build nginx based Alpine arm32v6 for Raspberry Pi/Orange Pi
To build Cachet for `arm32` a NGINX Alpine base image is needed. Since a base image `arm32v7/alpine` is not available, [arm32v6/alpine](https://hub.docker.com/r/arm32v6/alpine/) is used.  
Dockerfile: [nginx-alpine-arm32v6/Dockerfile](https://github.com/tedsluis/cachet-on-docker/blob/master/nginx-alpine-arm32v6/Dockerfile)
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/nginx:1.13.9-alpine-arm32v6 \
                                                    --file cachet-on-docker/nginx-alpine-arm32v6/Dockerfile
```
  
### Build Cachet based on NGINX Alpine arm32v6 for Raspberry Pi/Orange Pi
Dockerfile: [cachet/Dockerfile](https://github.com/tedsluis/cachet-on-docker/blob/master/cachet/Dockerfile)
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/cachet:2.3.13-nginx-alpine-arm32v6 \
                                                    --build-arg cachet_ver="v2.3.13" \
                                                    --file cachet-on-docker/cachet/Dockerfile
```
  
**More info**
ted.sluis@gmail.com  
https://hub.docker.com/u/tedsluis  
https://docs.cachethq.io  
https://github.com/CachetHQ/Cachet  
https://github.com/CachetHQ/Docker  
https://github.com/tedsluis/cachet-monitor  
https://castawaylabs.github.io/cachet-monitor  

