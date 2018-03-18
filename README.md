# cachet-on-docker
Cachet and Cachet-monitor running on Docker
  
Explained how to run it on raspbian (raspberry) or any other Linux distro like Fedora, Centos, RHEL, Debian, Ubuntu.  
  
## Install docker  
Use this way to install Docker on any linux distro:  
```
pi@raspberry:~ $ curl -sSL https://get.docker.com | sh
```
Note for raspberry: Docker runs nowadays on raspbian. It doesn't needed to use the [Hypriot Docker Image for Raspberry Pi](https://blog.hypriot.com/getting-started-with-docker-on-your-arm-device/) any more.  
More info at [https://docs.docker.com/install/](https://docs.docker.com/install/)  
  
## Create data directory for Postgres database  
By default containers use ephemeral storage. To ensure persistent storage a directory will be mounted into Postgres container.   
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
These unit files are configured to run on arm32v6 (like raspberry pi). If yu want to run these on X86 or amd64, you should change the Docker `image:tag` in these unit files:  
| Unit file                                                                                                   | arm32v6 Docker image                        | X86 / amd64 Docker image |
| ----------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------ |
| [docker-cachet.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-cachet.service)     | tedsluis/cachet:2.3.13-nginx-alpine-arm32v6 | cachethq/docker:latest   |
| [docker-postgres.service](https://github.com/tedsluis/cachet-on-docker/blob/master/docker-postgres.service) | arm32v6/postgres:alpine                     | postgres:9.5             |
| [cachet-monitor.service](https://github.com/tedsluis/cachet-on-docker/blob/master/cachet-monitor.service)   |                                             |                          |
  
## 
Simply copy these unit files from the root of the cloned repository to `/etc/systemd/system/` and do a `systemctl daemon-reload`:  
```
pi@raspberry:~/cachet-on-docker $ sudo cp *.service /etc/systemd/system
pi@raspberry:~/cachet-on-docker $ sudo systemctl daemon-reload
```
Finally you can `enable` these services, so they will always start after a reboot:   
```
pi@raspberry:~/cachet-on-docker $ sudo systemctl enable docker-postgres.service
pi@raspberry:~/cachet-on-docker $ sudo systemctl enable docker-cachet.service
```
  
## Build nginx based alpine arm32v6 for raspbian
note: used `arm32v6`, since base image `alpine arm32v7` not available
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/nginx:1.13.9-alpine-arm32v6 --file cachet-on-docker/nginx-alpine-arm32v6/Dockerfile
```
  
## Build cachet based on nginx alpine arm32v6 for raspbian
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/cachet:2.3.13-nginx-alpine-arm32v6 --build-arg cachet_ver="v2.3.13" --file cachet-on-docker/cachet/Dockerfile
```
  
