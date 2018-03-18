# cachet-on-docker
Cachet and Cachet-monitor running on Docker
  
Explained how to run it on `Raspbian` (boards like `Raspberry Pi` or `Orange Pi`) or any other Linux distros like `Fedora`, `Centos`, `RHEL`, `Debian`, `Ubuntu` and `Mint`.  
  
## Deploy Cachet, Postgres and Cachet-monitor as containers via systemd  
  
### Install docker  
One way to install Docker on any kind of Linux distro:  
```
pi@raspberry:~ $ curl -sSL https://get.docker.com | sh
```
More info at [https://docs.docker.com/install/](https://docs.docker.com/install/)  
Note for raspberry: Docker runs nowadays on raspbian. It isn't needed to use the [Hypriot Docker Image for Raspberry Pi](https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/) any more.  
  
### Create data directory for Postgres database  
By default containers use ephemeral storage. To ensure persistent storage, a directory will be mounted into the Postgres container.   
```
pi@raspberry:~ $ sudo mkdir -p /var/lib/progresql/data
```
  
### Clone this git repository  
This repo contains some configuration files that are needed to run Cachet.  
```
pi@raspberry:~ $ git clone https://github.com/tedsluis/cachet-on-docker.git
```  
   
### Init scripts  
Most Linux distros use `Systemd` manage services nowadays, so `Systemd` will be used to launch the containers. Each container has its own `unit file` (located in `/etc/systemd/system`). These `unit files` are configured to run on `arm32v6` (like `Raspberry Pi` or `Orange Pi`). If you want to run these on `X86` or `amd64`, you should change the Docker `image:tag` in each unit file:  

| Unit file                                                                                                   | arm32v6 Docker image                                                                                   | X86 / amd64 Docker image                                                               |
| :---------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------- |
| [docker-cachet.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-cachet.service)     | [tedsluis/cachet:2.3.13-nginx-alpine-arm32v6](https://hub.docker.com/r/tedsluis/cachet/)               | [cachethq/docker:latest](https://hub.docker.com/r/cachethq/docker/)                    |
| [docker-postgres.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-postgres.service) | [arm32v6/postgres:alpine](https://hub.docker.com/r/arm32v6/postgres/)                                  | [postgres:9.5](https://hub.docker.com/_/postgres/)                                     |
| [cachet-monitor.service](https://github.com/tedsluis/cachet-on-docker/blob/master/cachet-monitor.service)   | [tedsluis/cachet-monitor:2.3-golang-alpine-arm32v6](https://hub.docker.com/r/tedsluis/cachet-monitor/) | [castawaylabs/cachet-monitor:latest](https://hub.docker.com/r/castawaylabs/cachet-monitor/)   |
 
### Start services  
Simply copy these `unit files` from the root of the cloned repository to `/etc/systemd/system/` and do a `systemctl daemon-reload`:  
```
pi@raspberry:~ $ sudo cp *.service /etc/systemd/system
pi@raspberry:~ $ sudo systemctl daemon-reload
```
  
View the containers running:  
```
pi@raspberry:~ $ sudo docker ps
CONTAINER ID        IMAGE                                         COMMAND                  CREATED             STATUS              PORTS                          NAMES
0bb444ef9638        tedsluis/cachet:2.3.13-nginx-alpine-arm32v6   "/sbin/entrypoint.sh"    5 minutes ago       Up 1 minute         80/tcp, 0.0.0.0:80->8000/tcp   cachet
cb95d27600d6        arm32v6/postgres:alpine                       "docker-entrypoint.s…"   11 minutes ago      Up 1 minute         5432/tcp                       postgres
74f62aafc922        tedsluis/cachet-monitor:latest                "/bin/sh -c /app/cac…"   21 seconds ago      Up 1 minute                                        cachet-monitor
``` 
  
Finally you can `enable` these services, so they will always start after a reboot:   
```
pi@raspberry:~ $ sudo systemctl enable docker-postgres.service
pi@raspberry:~ $ sudo systemctl enable docker-cachet.service
pi@raspberry:~ $ sudo systemctl enable cachet-monitor.service
```
  
Use the commands below to view the logs or `stop`, `start` or `restart` the services:  
```
pi@raspberry:~ $ sudo journalctl -u <unit file>
pi@raspberry:~ $ sudo systemctl stop <unit file>
pi@raspberry:~ $ sudo systemctl start <unit file>
pi@raspberry:~ $ sudo systemctl restart <unit file>
```
  
## Build your own Cachet image for arm32
Unforunately for arm32 users (like Raspberry Pi and Orange Pi) there is no official `Cachet` image available. Users can pull my image `tedsluis/cachet:2.3.13-nginx-alpine-arm32v6` from Dockerhub, or they can build it them self as described below:  
  
### Build nginx based Alpine arm32v6 for Raspberry Pi/Orange Pi
To build Cachet for `arm32` a NGINX Alpine base image is needed. Since a base image `arm32v7/alpine` is not available, [arm32v6/alpine](https://hub.docker.com/r/arm32v6/alpine/) is used.  
Dockerfile: [cachet-on-docker/nginx-alpine-arm32v6/Dockerfile](https://github.com/tedsluis/cachet-on-docker/blob/master/nginx-alpine-arm32v6/Dockerfile)    
```
pi@raspberry:~ $ sudo docker build -t tedsluis/nginx:1.13.9-alpine-arm32v6 . \
                                              --file cachet-on-docker/nginx-alpine-arm32v6/Dockerfile

pi@raspberry:~ $ sudo docker images tedsluis/nginx
REPOSITORY          TAG                     IMAGE ID            CREATED             SIZE
tedsluis/nginx      1.13.9-alpine-arm32v6   5f9138b121b5        1 minute ago        15MB

```
  
### Build Cachet based on NGINX Alpine arm32v6 for Raspberry Pi/Orange Pi
Dockerfile: [cachet-on-docker/cachet/Dockerfile](https://github.com/tedsluis/cachet-on-docker/blob/master/cachet/Dockerfile)  
```
pi@raspberry:~ $ sudo docker build -t tedsluis/cachet:2.3.13-nginx-alpine-arm32v6 . \
                                              --build-arg cachet_ver="v2.3.13" \
                                              --file cachet-on-docker/cachet/Dockerfile

pi@raspberry:~ $ sudo docker images tedsluis/cachet
REPOSITORY          TAG                           IMAGE ID            CREATED             SIZE
tedsluis/cachet     2.3.13-nginx-alpine-arm32v6   53b5de7af0e5        1 minute ago        258MB 
```
  
### Build Cachet-monitor based on golang Alpine arm32v6 for Raspberry Pi/Orange Pi
For this build you need to clone the [https://github.com/tedsluis/cachet-monitor](https://github.com/tedsluis/cachet-monitor).  
Dockerfile: [cachet-monitor/Dockerfile](https://github.com/tedsluis/cachet-monitor/blob/master/Dockerfile)  
```
pi@raspberry:~ $ git clone https://github.com/tedsluis/cachet-monitor.git
pi@raspberry:~ $ sudo docker build -t tedsluis/cachet-monitor:latest . \
                                              --file cachet-monitor/Dockerfile 

sudo docker images tedsluis/cachet-monitor
REPOSITORY                TAG                         IMAGE ID            CREATED             SIZE
tedsluis/cachet-monitor   3.0-golang-alpine-arm32v6   b6db771b06ec        1 minutes ago      13.3MB
```
  
**More info**  
  
ted.sluis@gmail.com  
https://hub.docker.com/u/tedsluis  
https://docs.cachethq.io  
https://github.com/CachetHQ/Cachet  
https://github.com/CachetHQ/Docker  
https://github.com/tedsluis/cachet-monitor  
https://castawaylabs.github.io/cachet-monitor  

