### 1 - Redis Master Controller

* The first stage of launching the application is to start the Redis Master.
A Kubernetes service deployment has, at least, two parts. A replication controller and a service.

* The replication controller defines how many instances should be running, the Docker Image to use, and a name
to identify the service. Additional options can be utilized for configuration and discovery. 

> If Redis were to go down, the replication controller would restart it on an active node.

#### 1.1 - Create Replication Controller

In this example, the YAML defines a redis server called redis-master using the official redis running port 6379.

The kubectl create command takes a YAML definition and instructs the master to start the controller.

```
$ kubectl create -f redis-master-controller.yaml
```

**What's running?**

The above command created a Replication Controller. The Replication:

```
$ kubectl get rc
```

> All containers described as Pods. A pod is a collection of containers that makes up a particular application,
for example Redis. You can view this using:
```
$ kubectl get pods
```

### 2 - Redis Master Service

* The second part is a service. A Kubernetes service is a named load balancer that proxies traffic to one or
more containers. The proxy works even if the containers are on different nodes.

* ervices proxy communicate within the cluster and rarely expose ports to an outside interface.

> When you launch a service it looks like you cannot connect using curl or netcat unless you start it as
part of Kubernetes. The recommended approach is to have a LoadBalancer service to handle external communications.

#### 2.1 - Create Service

The YAML defines the name of the replication controller, redis-master, and the ports which should be proxied.
```
$ kubectl create -f redis-master-service.yaml
```

List & Describe Services:
```
$ kubectl get services
$ kubectl describe services redis-master
```

### 3 - Replication Slave Pods

* In this example we'll be running Redis Slaves which will have replicated data from the master.
More details of Redis replication can be found at http://redis.io/topics/replication

* As previously described, the controller defines how the service runs. In this example we need to determine
how the service discovers the other pods. The YAML represents the GET_HOSTS_FROM property as DNS.
You can change it to use Environment variables in the yaml but this introduces creation-order dependencies
as the service needs to be running for the environment variable to be defined.

#### 3.1 - Start Redis Slave Controller

In this case, we'll be launching two instances of the pod using the image kubernetes/redis-slave:v2.
It will link to redis-master via DNS.
```
$ kubectl create -f redis-slave-controller.yaml
```

_List Replication Controllers:_
```
$ kubectl get rc
```

### 4 - Redis Slave Service

As before we need to make our slaves accessible to incoming requests. This is done by starting a service which knows
how to communicate with redis-slave.

Because we have two replicated pods the service will also provide load balancing between the two nodes.

```
$ kubectl create -f redis-slave-service.yaml
$ kubectl get services
```

### 5 - Frontend Replicated Pods

With the data services started we can now deploy the web application. The pattern of deploying a web application
is the same as the pods we've deployed before.

* The YAML defines a service called frontend that uses the image _gcr.io/googlesamples/gb-frontend:v3.
The replication controller will ensure that three pods will always exist.
```
$ kubectl create -f frontend-controller.yaml
$ kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
frontend       3         3         3       11m
redis-master   1         1         1       16m
redis-slave    2         2         2       16m

$ kubectl get pods
```

**PHP Code**
The PHP code uses HTTP and JSON to communicate with Redis. When setting a value requests go to _redis-master_
while read data comes from the _redis-slave_ nodes.

### 6 - Guestbook Frontend Service

To make the frontend accessible we need to start a service to configure the proxy.

**Start Proxy**
The YAML defines the service as a _NodePort_. NodePort allows you to set well-known ports that are shared across
your entire cluster. This is like _-p 80:80_ in Docker.

In this case, we define our web app is running on port 80 but we'll expose the service on _30080_.

```
$ kubectl create -f frontend-service.yaml
$ kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
frontend       NodePort    10.101.115.225   <none>        80:30080/TCP   6m6s
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        97m
redis-master   ClusterIP   10.107.174.186   <none>        6379/TCP       15m
redis-slave    ClusterIP   10.100.3.142     <none>        6379/TCP       14m
```


### 7 - Access Guestbook Frontend

With all controllers and services defined Kubernetes will start launching them as Pods. A pod can have different
states depending on what's happening. For example, if the Docker Image is still being downloaded then the Pod will
have a _pending_ state as it's not able to launch. Once ready the status will change to _running_.

**View Pods Status**
You can view the status using the following command:
```
$ kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-8hjxr       1/1     Running   0          9m24s
frontend-msrph       1/1     Running   0          9m24s
frontend-rzwbv       1/1     Running   0          9m24s
redis-master-44hsn   1/1     Running   0          15m
redis-slave-9zrdf    1/1     Running   0          14m
redis-slave-npgq4    1/1     Running   0          14m
```

**Find NodePort**

> If you didn't assign a well-known NodePort then Kubernetes will assign an available port randomly.
You can find the assigned NodePort using kubectl.
```
$ kubectl describe service frontend | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  30080/TCP
```

**View UI**
Once the Pod is in running state you will be able to view the UI via port 30080. Use the URL to view the page
https://2886795272-30080-ollie02.environments.katacoda.com

> Under the covers the PHP service is discovering the Redis instances via DNS. You now have a working multi-tier
application deployed on Kubernetes.


