# cachet-on-docker
Cachet and Cachet-monitor running on Docker on raspbian / raspberry pi

## Install docker  
```
pi@raspberry:~ $ curl -sSL https://get.docker.com | sh
```
  
## Create data directory for Postgres database  
By default containers use ephemeral storage. To ensure persistent storage a directory will be mounted into Postgres container.   
```
pi@raspberry:~ $ sudo mkdir -p /var/lib/progresql/data
```
  
## Clone this git repo  
```
pi@raspberry:~ $ git clone https://github.com/tedsluis/cachet-on-docker.git
pi@raspberry:~ $ cd cachet-on-docker
```  
   
## Init scripts  
Systemd will be used to launch the containers. Each container has its own unit file:  
* cachet-monitor.service  
* docker-cachet.service  
* docker-postgres.service  
  
Simply copy these files to /etc/systemd/system/ and do a systemctl daemon-reload.  
Finally you can start and enable these services, so they will always start after a reboot:   
```
pi@raspberry:~/cachet-on-docker $ sudo cp *.service /etc/systemd/system
pi@raspberry:~/cachet-on-docker $ sudo systemctl daemon-reload
pi@raspberry:~/cachet-on-docker $ sudo systemctl start docker-postgres.service
pi@raspberry:~/cachet-on-docker $ sudo systemctl start docker-cachet.service 
pi@raspberry:~/cachet-on-docker $ sudo systemctl enable docker-postgres.service
pi@raspberry:~/cachet-on-docker $ sudo systemctl ensble docker-cachet.service
```
  
## Build nginx based alpine arm32v6 for raspbian
note: used arm6, since base image alpine arm32v7 not available
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/nginx:1.13.9-alpine-arm32v6 --file cachet-on-docker/nginx-alpine-arm32v6/Dockerfile
```
  
## Build cachet based on nginx alpine arm32v6 for raspbian
```
pi@raspberry:~/cachet-on-docker $ sudo docker build -t tedsluis/cachet:2.3.13-nginx-alpine-arm32v6 --build-arg cachet_ver="v2.3.13" --file cachet-on-docker/cachet/Dockerfile
```
  
