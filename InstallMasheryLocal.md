

# MASHERY Local 5.3 - Installation guide
 

## Docker installation
* sudo yum update
* sudo yum install epel-release	
* sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
* sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
* sudo yum install -y yum-utils device-mapper-persistent-data lvm2 bash-completion jq
* sudo yum install docker-ce docker-ce-cli containerd.io

* sudo systemctl enable docker
* sudo systemctl start docker

## Add user to run docker without sudo 
* sudo groupadd docker
* sudo newgrp docker
* sudo usermod -aG docker $USER

## Test it !
* sudo docker images
* docker ps
* docker info

## docker-compose install
* sudo yum install docker-compose
* sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-\$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
* sudo chown mashery:mashery /usr/local/bin/docker-compose 
* sudo chmod +x /usr/local/bin/docker-compose 

## Docker Registry setup
* docker pull registry
* docker run -d -p 5000:5000 --restart always --name registry registry:2

## Authorize access to Docker Registry from docker-machine
* docker-machine ssh default
* vi /etc/docker/daemon.json
	Add this line : {"insecure-registries":["192.168.99.100:5000"]}
* Restart docker machine : docker-machine restart default
## Create swarm cluster
* Swarm init
> docker swarm init --advertise-addr 192.168.99.100
* list of nodes : 
> docker node ls
* Login to registry within SWARM cluster : 
> docker login 192.168.99.100:5000
* Swarm vizualization app :
> docker service create \
--name=swarm-viz \
--publish=3000:3000 \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
mikesir87/swarm-viz

## Portainer
### using docker commands
Install Portainer
> docker pull portainer/portainer:latest

> docker volume create portainer_data

> docker run -d \
 --name portainer-01 \
 --restart unless-stopped \
 -p 9000:9000 \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v portainer_data:/data \
 portainer/portainer
 
### using docker stack
> curl -L https://downloads.portainer.io/agent-stack.yml -o agent-stack.yml && docker stack deploy --compose-file=agent-stack.yml portainer-agent


# Mashery installation
## TML_INSTALLER creation
* cd TIB_Mash_local_5.3.0/tml-quickstart/
* ./quick-start.sh -b false -d false

## Fix Maven issue : 
> https://support.tibco.com/s/article/How-to-fix-Maven-host-for-ML-5-3-0-or-lower-and-ML-5-3-0-HF1
* chmod +x fix_maven_host_for_TML_5.3.0.sh 
* docker cp fix_maven_host_for_TML_5.3.0.sh tml-installer:/root
* docker exec -it -u root tml-installer /root/fix_maven_host_for_TML_5.3.0.sh

## Launch TML Installer
* cd Downloads/TIB_Mash_local_5.3.0/
* ./start-installer 192.168.99.100
    > get the ip of the docker machine : docker-machine env default

## Open Jenkins
http://192.168.99.100:8080

In Jenkins :
* Build > build_docker
* Prepush > configure_docker_registry_v2
    *   DOCKER_REGISTRY_HOST : localhost
 	*   DOCKER_REGISTRY_PORT : 5000
	*   DOCKER_REGISTRY_USERNAME : none
 	*   DOCKER_REGISTRY_PASSWORD : none
 	*   DOCKER_REGISTRY_REPO : tml
* Push > push_docker_to_docker_registry_v2
* Deploy > update_platform_api_properties
* Deploy > build_deployment_package



# Deploy MASHERY Cluster
* Generate config files
> cd docker-deploy/onprem/quick-start
> ./compose.sh manifest-onprem-swarm.json
* Correct Compose files

> sed -i 's/${HOST_NAME}/default/g' *.yml


* Create Cassandra container
docker stack deploy -c tmgc-nosql.yml nosqlstack --with-registry-auth
* Create ClusterManager
docker stack deploy -c tmgc-cm.yml cmstack --with-registry-auth
* Create ogger container
docker stack deploy -c tmgc-log.yml logstack --with-registry-auth
* Create SQL container
docker stack deploy -c tmgc-sql.yml sqlstack --with-registry-auth
* Create cache container
docker stack deploy -c tmgc-cache.yml cachestack --with-registry-auth
* Check all services are started
docker service ls && docker ps
* Deploy TML
./deploy-tm-pod.sh 

* docker service ls
ID            NAME                   MODE        REPLICAS  IMAGE
jfqyzzr2i2bk  cachestack_cache       replicated  1/1       tml-cache:v5.3.0.HF1.3
k2uuiqeii61n  sqlstack_sql           replicated  1/1       tml-sql:v5.3.0.HF1.3
k5jjx0f8t4pp  logstack_log           replicated  1/1       tml-log:v5.3.0.HF1.3
qz1fwo63vmxm  tmstack_tm             replicated  1/1       tml-tm:v5.3.0.HF1.3
s4hqt0eb2826  nosqlstack_nosql_seed  replicated  1/1       tml-nosql:v5.3.0.HF1.3
vx64pp9i47ld  cmstack_tmlcm          replicated  1/1       tml-cm:v5.3.0.HF1.3

# Validate the install
> docker ps -a

Use the tml-cm container id to connect

> docker exec -it 2927ff95d59b bash

And run:

> cm ls components

> cm cluster status

> curl http://<COMPONENT_HOST_IP>:9080/container/status

## Access the ML install:
* Developer Portal: https://<HOST>:10443/
* Configuration Manager: https://<HOST>:10443/admin

* UserName: admin
* Password: Ap1Us3rPasswd

## OPTIONAL : docker machine
### docker-machine install
* sudo curl -L "https://github.com/docker/machine/releases/download/v0.14.0/docker-machine-\$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-machine
* sudo chmod +x /usr/local/bin/docker-machine 
* docker-machine version
* sudo wget https://raw.githubusercontent.com/docker/machine/master/contrib/completion/bash/docker-machine-prompt.bash -O /etc/bash_completion.d/docker-machine-prompt.bash
* sudo wget https://raw.githubusercontent.com/docker/machine/master/contrib/completion/bash/docker-machine.bash -O /etc/bash_completion.d/docker-machine.bash
* docker-machine ls
### create docker machine
* sudo wget https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo -P /etc/yum.repos.d
* sudo yum install kernel-devel kernel-headers make patch gcc
* sudo yum install VirtualBox-5.2
* docker-machine create -d virtualbox --virtualbox-disk-size "40000" default
* eval $(docker-machine env default)

### Edit config of docker-machine
* VBoxManage modifyvm default --memory 4096
* VBoxManage modifyvm default --cpus 2
* VBoxManage showvminfo default*

## OPTIONAL : Secured Registry within swarm

* Create certificate
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout registry.key -x509 -days 365 \
  -out registry.crt

* Push it to the docker swarm master
docker-machine scp registry.crt master:/home/docker/ && \
docker-machine ssh master sudo mkdir -p /etc/docker/certs.d/registry.swarm.localdomain:5000 && \
docker-machine ssh master sudo mv /home/docker/registry.crt /etc/docker/certs.d/registry.swarm.localdomain:5000/ca.crt

* Do the same for node-1
docker-machine scp registry.crt node-1:/home/docker/ && \
docker-machine ssh node-1 sudo mkdir -p /etc/docker/certs.d/registry.swarm.localdomain:5000 && \
docker-machine ssh node-1 sudo mv /home/docker/registry.crt /etc/docker/certs.d/registry.swarm.localdomain:5000/ca.crt

* add registry.swarm.localdomain domain to /etc/hosts
docker-machine ssh master
sudo sh -c "echo ' 192.168.99.101 registry.swarm.localdomain' >> /etc/hosts"
exit

> do the same command for node-1 : docker-machine ssh node-1

* Create registry in HTTPS mode																			
	* copy certs to master node
docker-machine scp registry.crt master:/home/docker/ && \
docker-machine scp registry.key master:/home/docker/

	* create registry service
docker service create --name registry --publish=5000:5000 \
 --constraint=node.role==manager \
 --mount=type=bind,src=/home/docker,dst=/certs \
 -e REGISTRY_HTTP_ADDR=0.0.0.0:5000 \
 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.crt \
 -e REGISTRY_HTTP_TLS_KEY=/certs/registry.key \
 registry:latest
* et voilà

# Useful links:
> https://docs.tibco.com/pub/mash-local/5.3.0/doc/html/GUID-295E7EE4-4CE5-491F-964E-2BE786C21A39-homepage.html

> https://docs.tibco.com/pub/mash-local/5.3.0/doc/html/GUID-295E7EE4-4CE5-491F-964E-2BE786C21A39-homepage.html

> https://stackoverflow.com/questions/17157721/how-to-get-a-docker-containers-ip-address-from-the-host

> https://docs.docker.com/machine/reference/create/

> https://esalagea.wordpress.com/2016/01/21/start-a-docker-container-on-centos-at-boot-time-as-a-linux-service/

> https://codeblog.dotsandbrackets.com/private-registry-swarm/

> https://github.com/mikesir87/swarm-viz
> https://stackoverflow.com/questions/37599128/docker-how-do-you-disable-auto-restart-on-a-container

> https://www.portainer.io/installation/


### ALIAS
alias cm="docker exec -it \`docker ps | grep cmstack_tmlcm | cut -d " " -f1\`  /usr/local/bin/clustermanager"

# Login into Configuration Manager:
UserName: 
> admin

Password: 
> Ap1Us3rPasswd
																									
