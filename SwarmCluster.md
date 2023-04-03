# Cloud Servers Configuration

Creating a cloud server to perform cloud orchestration using Docker Swarm.
Making it short and concise to see the example use cases.


## Server
`Using Virtual Machine with linux ubuntu`
- OS - Ubuntu (Virtual Machine image)

Install it or use any pre-installed linux OS.

> **Note**:
> Use bridged adapter before starting machine. 

![netwok setting](https://user-images.githubusercontent.com/30742340/229444004-e7a2dee4-cdbb-41f9-92fa-c11010ac142f.PNG)

## Prerequisite
- Docker (to install see [Docker](https://docs.docker.com/engine/install/ubuntu/#set-up-the-repository))
- Linux knowledge
- Networking


Verify docker service by checking version:
```bash
  docker -v
```
if unable to execute docker as non-root user, use this command it will add user to docker group  
```bash
sudo usermod -aG docker <USERNAME>
```


### `Frequently used commands`
if you're familier with docker commands 
```bash
   docker ps -a 
   docker rm <name/id>
   docker volume ls
   docker network ls
   docker inspect <name/id>
   docker rmi imagename
   docker pull image:version
```


## Nodes(servers) configuration

Using docker inside docker so we can mimic a cloud of multiple nodes as much as we want.
we will extend servers later.

&nbsp;
# Create New Network

I'm using bridge network because i want to access my services in host machine (windows) 

```bash
docker network create -d bridge newnetwork
```
we are performing this step to make a connection between nodes.

In AWS/GCP or Any cloud provider there is no need to create a network cause we can access them with there public IP.


## Creating 4 servers
Check internet connectivity by

```bash
ping google.com
```
commands to run docker:dind container (docker:dind will act like a server)

#### MASTER Server
```bash
docker run --privileged -dit --network newnetwork -p 7000-7100:7000-7100 --name MASTER docker:dind
```
#### WORKER 1 Server
```bash
docker run --privileged -dit --network newnetwork --name WORKER1 docker:dind
```
#### WORKER 2 Server
```bash
docker run --privileged -dit --network newnetwork --name WORKER2 docker:dind
```
#### WORKER 3 Server
```bash
docker run --privileged -dit --network newnetwork --name WORKER3 docker:dind
```
 
#### commands args explaination:

- `--privileged` : giving host machine abilities to container, without this we docker:dind will not going to work
- `-d` : running in detached mode
- `-it` : shows output of containers terminal
- `-p` : opening ports on master server
- `--network` : to use available networks.


These four commands will create 4 server.

On master server we are opening 100 ports so we can use them for diffrent services that help us to build fully automatted server orchestration.
Opening port is optional for production

&nbsp;
# Docker SWARM

## Initializing docker swarm
To initialize docker swarm on master node/server we need IPAddress / publicIP of master server. 

`cmd to see the IP of container server`
```bash
docker inspect MASTER | grep -E "IPAddress"
```
#### output
![output](https://user-images.githubusercontent.com/30742340/229463236-ce480e7d-65ec-4d0f-a797-dfe529ffaee4.PNG)


### Two ways to use master server
- execute cmd from outside of docker 
```bash
docker exec -it MASTER (command here)
```
- attach to docker shell
```bash
docker exec -it MASTER sh
```

&nbsp;
## SWARM init
we are doing it by attach method.
Pay attention while executing commands first always attach to master server container before executing CMDs.

use `docker exec -it MASTER sh` command to get a shell of master server.



#### RUN
```bash
docker swarm init --advertise-addr <masterIP>
```
Above command will create a swarm cluster and give a genrated token to join the swarm cluster.
either use this token or genrate a new one with this:

```bash
docker swarm join-token worker
```

On base machine
![token](https://user-images.githubusercontent.com/30742340/229468227-cd5f8b36-e9cf-4c42-b0dd-27a85d1b7435.PNG)

&nbsp;
Inside MASTER server container
![incontainer](https://user-images.githubusercontent.com/30742340/229471936-8d9a5d45-d78b-4d25-a884-42c23b92a9f2.PNG)

&nbsp;

### To see available nodes in swarm cluster

```bash
 docker node ls 
```

## Adding other servers to swarm cluster
use given token command for all other workers/containers nodes

running from base machine.
Execute this command for every conainer to join as a worker.
```bash
docker exec -it WORKER1 docker swarm join --token SWMTKN-1-07m0j52n(fulltoken)

docker exec -it WORKER2 docker swarm join --token SWMTKN-1-07m0j52n(fulltoken)

docker exec -it WORKER3 docker swarm join --token SWMTKN-1-07m0j52n(fulltoken)
```


Now we have configured a 4 server/nodes swarm cluster 
1 node is leader and manager(MASTER node)
3 nodes are workers.
 
`see swarm cluster again`

```bash
docker exec -it MASTER docker node ls
```
All available servers

![output](https://user-images.githubusercontent.com/30742340/229477171-3cde496e-ee7d-4a53-8f56-8fa7d0857fbc.PNG)

Now we will use this cluster for our services and applications.
