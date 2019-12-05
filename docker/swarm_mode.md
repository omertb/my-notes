(source: https://www.katacoda.com/courses/docker-production)
### 1 - Create Swarm Mode Cluster

Swarm Mode is built into the Docker CLI. You can find an overview the possibility commands via `docker swarm --help

The most important one is how to initialise Swarm Mode. Initialisation is done via init.
```
$ docker swarm init
```

### 2 - Join Cluster

> For demonstration purposes, we'll ask the manager what the token is via swarm join-token. In production, this token
should be stored securely and only accessible by trusted individuals.
```Shell
$ token=$(ssh -o StrictHostKeyChecking=no 172.17.0.43 "docker swarm join-token -q worker") && echo $token

```

* On the second host, join the cluster by requesting access via the manager. The token is provided as an additional
parameter.

```
$ docker swarm join 172.17.0.43:2377 --token $token
```

* By default, the manager will automatically accept new nodes being added to the cluster. You can view all nodes
in the cluster using `docker node ls

### 3 - Create Overlay Network

> The _overlay_ network is used to enable containers on different hosts to communicate. Under the covers, this is
a Virtual Extensible LAN (VXLAN), designed for large scale cloud based deployments.

* The following command will create a new overlay network called skynet. All containers registered to this network
can communicate with each other, regardless of which node they are deployed onto.
```
$ docker network create -d overlay skynet
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
ee624b0c509f        bridge              bridge              local
90fa7c3e1cfa        docker_gwbridge     bridge              local
f46cc1338e5a        host                host                local
u4lrr9nk8o2e        ingress             overlay             swarm
9e826c05fcfd        none                null                local
kcwbjmnpdssn        skynet              overlay             swarm
```

### 4 - Deploy Service

> By default, Docker uses a spread replication model for deciding which containers should run on which hosts.
The spread approach ensures that containers are deployed across the cluster evenly. This means that if one of
the nodes is removed from the cluster, the instances would be already running on the other nodes. The workload
on the removed node would be rescheduled across the remaining available nodes.

> A new concept of Services is used to run containers across the cluster. This is a higher-level concept than
containers. A service allows you to define how applications should be deployed at scale. By updating the service,
Docker updates the container required in a managed way.

1. In this case, we are deploying the Docker Image katacoda/docker-http-server. We are defining a friendly name of
a service called http and that it should be attached to the newly created skynet network.

2. For ensuring replication and availability, we are running two instances, of replicas, of the container across
our cluster.

3. Finally, we load balance these two containers together on port 80. Sending an HTTP request to any of the nodes
in the cluster will process the request by one of the containers within the cluster. The node which accepted
the request might not be the node where the container responds. Instead, Docker load-balances requests across
all available containers.

```
docker service create --name http --network skynet --replicas 2 -p 80:80 katacoda/docker-http-server
```

* You can view the services running on the cluster using the CLI command `docker service ls

> As containers are started you will see them using the ps command. You should see one instance of the container on each host.

* List containers on the first host - `docker ps`

* List containers on the second host - `docker ps`

> If we issue an HTTP request to the public port, it will be processed by the two containers  
```
$ curl host01
<h1>This request was processed by host: 398937d9c040</h1>
$ curl host01
<h1>This request was processed by host: 1cc613ba3365</h1>
$ curl host01
<h1>This request was processed by host: 777512419e14</h1>
$ curl host01
<h1>This request was processed by host: 69bc0807a4d8</h1>
$ curl host01
<h1>This request was processed by host: 398937d9c040</h1>
```

### 5 - Inspect State

> You can view the list of all the tasks associated with a service across the cluster. In this case, each task is
a container:
```
docker service ps http
```

> You can view the details and configuration of a service via:

```
docker service inspect --pretty http
```

> On each node, you can ask what tasks it is currently running. Self refers to the manager node Leader:
```
docker node ps self
```

> Using the ID of a node you can query individual hosts:
```
docker node ps $(docker node ls -q | head -n1)
```

### 6 - Scale Service

The command below will scale our http service to be running across five containers.
```
docker service scale http=5
```

On each host, you will see additional nodes being started `docker ps`