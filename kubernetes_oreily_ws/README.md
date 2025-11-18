# Kubernetes in 4 Hours

## Description

This is a workshop by Sander (mail@sandervanvugt.nl) from Oreily on 2025 November 17. UTC 3:00pm. 

### Descriptions

1. Rootless and Rootful

- Rootless containers are designed to run without requiring root privileges on the host system. Meaning that users can create, manage, and run containers without needing administrative rights.
- Rootful containers, on the other hand, require root privileges to run. This means that the container engine and the processes inside the container operate with elevated permissions.

2. Stateless and Stateful applications

- Stateful applications retain information about user interactions and maintain session data on the server, allowing for a continuous experience, like online banking.
- In contrast, stateless applications treat each request independently without storing any session information, making them more scalable and efficient, like a vending machine.

3. Microservices

- Microservices are an architectural style for developing software applications as a collection of small, independent services that communicate with each other over a network.
- Each service focuses on a specific business capability and can be developed, deployed, and scaled independently, allowing for greater flexibility and faster development cycles.


```bash
# I have aliasses k='kubectl'; in my bashrc

# list all resources
kubectl get all

# creates a deployment
k8 create deploy webapp --image=nginx --replicas=3

# list current active deployments
k get deployments

# delete a deployment
k delete deployment webapp

# kubernetes quotas
k create quota -h | less

```

## Kubernetes Resource Types

- **Pods**: The basic unit in Kubernetes, represents typically one and sometimes more containers that share common resources.

- **Deployments**: The application itself, standard entity that is rolled out with Kubernetes.

- **Services**: Make deployments accessible from the outside by providing a single IP/port combination.

- **Persistant Volumes**: Persistent (networked) storage.

- **ConfigMaps**: Allow for storing configuration and other specific parameters in a cloud environment.


```bash
# the system services in kube-system namespace
k get pods -n kube-system
```


>[!NOTE]
> Have seperate pods for tiered applications 1 for frontend, 1 for backend, 1 for data

>[!IMPORTANT]
> Do not run standalone pods, run deployments



## Managing apps with kubectl

```bash
k get all --selector app=webapp
NAME                         READY   STATUS    RESTARTS   AGE
pod/webapp-c89577dd5-bpszn   1/1     Running   0          15m
pod/webapp-c89577dd5-gsdrf   1/1     Running   0          15m
pod/webapp-c89577dd5-vs92c   1/1     Running   0          15m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp   3/3     3            3           15m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-c89577dd5   3         3         3       15m

# ------------------------------------------------------------------------

k get all --selector app=dashapp
NAME                           READY   STATUS    RESTARTS   AGE
pod/dashapp-8578ff4d9f-66cbp   1/1     Running   0          25s
pod/dashapp-8578ff4d9f-9fb27   1/1     Running   0          25s
pod/dashapp-8578ff4d9f-bhzkb   1/1     Running   0          25s
pod/dashapp-8578ff4d9f-qj2lm   1/1     Running   0          25s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashapp   4/4     4            4           25s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/dashapp-8578ff4d9f   4         4         4       25s
```

```bash
# scaled down from 4 to 2
k scale deploy dashapp --replicas=2

#describe 
#Events are what happened since the start of the resource.

```

## Using kubectl in a declarative way

- To work with Kubernetes the DevOps way, you should define the desired configuration in a YAML manifest file
- This declarative methodology is giving you much more control than the imperative methodology where you create all from the CLI
- After defining the desired state, use `kubectl apply -f myfile.yaml` to add the configuration to the cluster or change an existing resource

```bash
# generate the yaml manifest file
k get deploy webapp -o yaml > kube-repo/kubernetes_oreily_ws/deployment_webapp.yaml
k get pod webapp-c89577dd5-bpszn -o yaml > kube-repo/kubernetes_oreily_ws/pod_webapp.yaml

# from dry run
k create deploy mynginx --image=nginx --dry-run=client -o yaml > kube-repo/kubernetes_oreily_ws/deployment_mynginx.yaml

# add the generated configs to cluster
k apply -f myfile.yaml

# get info
k explain pods.spec


k get ns
NAME              STATUS   AGE
default           Active   73m
kube-node-lease   Active   73m
kube-public       Active   73m
kube-system       Active   73m

k get all -n <name_space>
```


>[!IMPORTANT]
    ```bash
    k create mydb --image=mariadb --replicas=3
    # create need to specify the resource type
    # for pods ==> k run
    ```

## Troubleshooting basic infra

```bash
k create deploy mydb --image=mariadb --replicas=3

k describe mydb-5fcf4f6fbb-s4nmp
k logs mydb-5fcf4f6fbb-s4nmp

2025-11-17 17:32:42+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:12.0.2+maria~ubu2404 started.
2025-11-17 17:32:42+00:00 [Warn] [Entrypoint]: /sys/fs/cgroup///memory.pressure not writable, functionality unavailable to MariaDB
2025-11-17 17:32:42+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2025-11-17 17:32:42+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:12.0.2+maria~ubu2404 started.
2025-11-17 17:32:43+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
        You need to specify one of MARIADB_ROOT_PASSWORD, MARIADB_ROOT_PASSWORD_HASH, MARIADB_ALLOW_EMPTY_ROOT_PASSWORD and MARIADB_RANDOM_ROOT_PASSWORD
```

## How to pass env variables

```bash
k set env deploy/mydb MARIADB_ROOT_PASSWORD=password

k get all --selector app=mydb
NAME                        READY   STATUS              RESTARTS       AGE
pod/mydb-5fcf4f6fbb-85zgj   0/1     CrashLoopBackOff    5 (101s ago)   5m27s
pod/mydb-5fcf4f6fbb-s4nmp   0/1     CrashLoopBackOff    5 (111s ago)   5m27s
pod/mydb-6447978996-kdv8g   1/1     Running             0              7s
pod/mydb-6447978996-rqgfc   0/1     ContainerCreating   0              2s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mydb   1/3     2            1           5m28s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/mydb-5fcf4f6fbb   2         2         0       5m27s
replicaset.apps/mydb-6447978996   2         2         1       7s

# the old replica set will be remained but deprecated -- check later
```

```bash
# how to execute commands in a pod/container
k exec -it mydb-6447978996-kdv8g -- /bin/sh
```

## How to expose services

- *ClusterIP* is accessible from within the cluster only.
- *NodePort* exposes an external port on the cluster nodes, thus providing a primitive way for offering access to the services.

`expose ---> service expose`

- *Ingress* is what should be used to provide user-friendly access to HTTP/HTTPS-based services.


```bash
# expose service
kubectl expose deployment nginxsvc --port=80

kubectl describe svc nginxsvc # look for endpoints

# check service
kubectl get svc

# check for endpoints
kubectl get endpoints

# get more data on the resources
k get all --selector app=nginxsvc -o wide
```

```bash

                                  |---> pod1
                                  |
user ---> ingress ---> service ---|---> pod2
                                  |
                                  |---> pod3

-------------------------------------------------------------

                    32000       |-node1-|                         |- pod1
                                |       |                         |
user    NodePort    32000       |-node2-|--------Service----------|- pod2
                                |       |       (ClusterIP)       |
                    32000       |-node3-|                         |- pod3

                                (clusternetwork)

```

- When executing `curl` to get the response from the nginxsvc service, I had to do some extra work since I'm using docker-desktop built-in Kubernetes.
- The *ClusterIP* is only accessible within the cluster.

```bash
k get svc nginxsvc
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
nginxsvc   ClusterIP   10.105.178.121   <none>        80/TCP    40m

# created another pod to run busybox
k run curlpod --rm -it --image=busybox --restart=Never -- sh

wget -qO- http://nginxsvc:80
```
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

- But if we expose the service with `--type=NodePort` we can expose the service to the host network.
- Running `curl http:localhost:<NodePort>` should return the response.

```bash
k delete svc nginxsvc

k expose deployment nginxsvc --port=80 --type=NodePort

k get svc nginxsvc
NAME       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginxsvc   NodePort   10.111.175.114   <none>        80:32064/TCP   3s
```
```html
curl http://localhost:32064
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```


## Storage

- Two types of volumes available. Ephemeral and Persistant.
- PersistentVolume is a resource in the cluster just like a node is a cluster resource.
- A PersistentVolumeClaim (PVC) is a request for storage by a user.



## ConfigMaps



```bash
# create a config map
k create cm myconf -h | less

k create cm mynewdbvars --fromliteral=MARIADB_ROOT_PASSWORD=password

# how the config specs indicates the variables are once set to get from config map
    spec:
      containers:
      - env:
        - name: MARIADB_ROOT_PASSWORD
          valueFrom:
            configMapKeyRef:
              key: MARIADB_ROOT_PASSWORD
              name: mynewdbvars

k get cm -n kube-systems
```


## Ingress

dns ---> ingress controller on cluster ---> ingress resource ---> service --> any

```bash
# create a rule to forward traffic
kubectl create ing nginxsvc --rule="myapp.info/=nginxsvc:80"
```


