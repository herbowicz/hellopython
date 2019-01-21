# hellopython

https://hub.docker.com/r/herbowicz/hellopython


## Display Docker version and info
docker --version
docker version
docker info

## Execute existing Docker image
docker run hello-world


## Use Dockerfile to build image
docker build --tag=friendlyhello .

## Run the app in detached mode on port 4000 
docker run -d -p 4000:80 friendlyhello

## List Docker images
docker image ls

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398

## List Docker containers (running / all)
docker ps / docker ps -a
docker container ls / docker container ls --all

CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago

## Stop container
docker container stop 1fa4ab2cf395

## Remove all images
docker rmi $(docker images -a -q)


## Login to Docker Hub
docker login

## Tag your image
docker tag image username/repository:tag
##example: docker tag friendlyhello herbowicz/friendlyhello

## Upload tagged image to the repository
docker push herbowicz/friendlyhello

## Execute newly created Docker image
docker run herbowicz/friendlyhello


## Swarmes
docker swarm init

## Run load-balanced app
docker stack deploy -c docker-compose.yml hellopython
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file

docker stack ls                                            # List stacks or apps
NAME                SERVICES            ORCHESTRATOR
hellopython         1                   Swarm

docker service ls                 # List running services associated with an app
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
kbkh257gvr29        hellopython_web     replicated          3/3                 herbowicz/hellopython:latest   *:4000->80/tcp

## List services
docker ps
docker container ls -q                                      # List container IDs
docker service ps <service>                  # List tasks associated with an app
$ docker service ps hellopython_web
ID                  NAME                IMAGE                          NODE                    DESIRED STATE       CURRENT STATE           ERROR               PORTS
xr7z6ba997lj        hellopython_web.1   herbowicz/hellopython:latest   linuxkit-00155dc44203   Running             Running 5 minutes ago
7hyg5wvo7a0u        hellopython_web.2   herbowicz/hellopython:latest   linuxkit-00155dc44203   Running             Running 5 minutes ago
rwkhvxhjwyka        hellopython_web.3   herbowicz/hellopython:latest   linuxkit-00155dc44203   Running             Running 5 minutes ago

docker inspect <task or container>                   # Inspect task or container
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager

## View nodes in swarm
docker node ls

docker stack ps hellopython
docker service ls


## Using other registry 
docker login quay.io
docker stack deploy --with-registry-auth -c docker-compose.yml hellopython


# Swarmes using VM
## Create virtual machines
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

## List machines
docker-machine ls

NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.101:2376           v18.09.1
myvm2   -        virtualbox   Running   tcp://192.168.99.102:2376           v18.09.1

## Make the first node as a manager
docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.101:2376"

## The second node joins as a worker
docker-machine ssh myvm2 "docker swarm join --token SWMTKN-1-3r4f28av03cizue9jcv2m9lsuq1c8vlz3vlltnhy1fminc4tl2-aolz8zrqpc00o301tw35epe58 192.168.99.101:2377"

## Set env variable to the first node
eval $(docker-machine env myvm1)

## Re-run stack deploy
docker stack deploy -c docker-compose.yml hellopython

## List machines (node with star is the manager)
docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
p5yeuwqxtr72vq4dtfg5nfxcp *   myvm1               Ready               Active              Leader              18.09.1
dxb3uk06og030q738jo3z5jf3     myvm2               Ready               Active                                  18.09.1


## Create space for redis on the manager
docker-machine ssh myvm1 "mkdir ./data"

192.168.99.101:8080 - Visualizer
192.168.99.101 - the app


## Verify running services
docker service ls
ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
x7uij6xb4foj        getstartedlab_redis        replicated          1/1                 redis:latest                      *:6379->6379/tcp
n5rvhm52ykq7        getstartedlab_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
mifd433bti1d        getstartedlab_web          replicated          5/5                 gordon/getstarted:latest    *:80->80/tcp
