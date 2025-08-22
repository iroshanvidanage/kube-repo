# Resolve VolumeMounts Issue in Kubernetes


Check the config map folder and other contianer manifest files to see if the shared folder is the same..

```bash
# the config-map shared folder
    root /var/www/html;

# the php-container shared folder
    Mounts:
      /usr/share/nginx/html from shared-files (rw)

# the nginx-container shared folder
    Mounts:
      /etc/nginx/nginx.conf from nginx-config-volume (rw,path="nginx.conf")
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cdprn (ro)
      /var/www/html from shared-files (rw)

```

---


```bash

thor@jumphost ~$ k get all
NAME               READY   STATUS    RESTARTS   AGE
pod/nginx-phpfpm   2/2     Running   0          33s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          25m
service/nginx-service   NodePort    10.96.141.157   <none>        8099:30008/TCP   18m

```


```bash

thor@jumphost ~$ k describe configmap/nginx-config -n default
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx.conf:
----
events {
}
http {
  server {
    listen 8099 default_server;
    listen [::]:8099 default_server;

    # Set nginx to serve files from the shared volume!
    root /var/www/html;
    index  index.html index.htm index.php;
    server_name _;
    location / {
      try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
      include fastcgi_params;
      fastcgi_param REQUEST_METHOD $request_method;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_pass 127.0.0.1:9000;
    }
  }
}


BinaryData
====

Events:  <none>

```

```bash

thor@jumphost ~$ k describe pods/nginx-phpfpm -n default
Name:             nginx-phpfpm
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Fri, 22 Aug 2025 11:44:29 +0000
Labels:           app=php-app
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  php-fpm-container:
    Container ID:   containerd://e25c6ef3e836c46cf782ce7cb51c358e3da53bc0b7c8836a307ffc8c31b9eee3
    Image:          php:7.2-fpm-alpine
    Image ID:       docker.io/library/php@sha256:2e2d92415f3fc552e9a62548d1235f852c864fcdc94bcf2905805d92baefc87f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 22 Aug 2025 11:44:34 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from shared-files (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cdprn (ro)
  nginx-container:
    Container ID:   containerd://97fd28fa6791c04e3ec733e085566118e461a074b36065fcdef8ae22b5f1df83
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:33e0bbc7ca9ecf108140af6288c7c9d1ecc77548cbfd3952fd8466a75edefe57
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 22 Aug 2025 11:44:42 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/nginx/nginx.conf from nginx-config-volume (rw,path="nginx.conf")
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cdprn (ro)
      /var/www/html from shared-files (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  shared-files:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  nginx-config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      nginx-config
    Optional:  false
  kube-api-access-cdprn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m47s  default-scheduler  Successfully assigned default/nginx-phpfpm to kodekloud-control-plane
  Normal  Pulling    3m46s  kubelet            Pulling image "php:7.2-fpm-alpine"
  Normal  Pulled     3m42s  kubelet            Successfully pulled image "php:7.2-fpm-alpine" in 3.914715855s (3.914739791s including waiting)
  Normal  Created    3m42s  kubelet            Created container php-fpm-container
  Normal  Started    3m42s  kubelet            Started container php-fpm-container
  Normal  Pulling    3m42s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     3m34s  kubelet            Successfully pulled image "nginx:latest" in 7.536313499s (7.536331132s including waiting)
  Normal  Created    3m34s  kubelet            Created container nginx-container
  Normal  Started    3m34s  kubelet            Started container nginx-container

```

```bash

thor@jumphost ~$ kubectl get pod/nginx-phpfpm -o yaml > pod.yml
thor@jumphost ~$ vi pod.yml 
thor@jumphost ~$ 
thor@jumphost ~$ 
thor@jumphost ~$ 
thor@jumphost ~$ k replace -f pod.yml --force
pod "nginx-phpfpm" deleted
pod/nginx-phpfpm replaced
thor@jumphost ~$ 

```

```bash

thor@jumphost ~$ k logs -f nginx-phpfpm
Defaulted container "php-fpm-container" out of: php-fpm-container, nginx-container
[22-Aug-2025 12:02:27] NOTICE: fpm is running, pid 1
[22-Aug-2025 12:02:27] NOTICE: ready to handle connections
^C
thor@jumphost ~$ k logs -f nginx-phpfpm nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2025/08/22 12:02:31 [error] 80#80: *1 directory index of "/var/www/html/" is forbidden, client: 10.244.0.1, server: _, request: "GET / HTTP/1.1", host: "30008-port-dkybeqsaea46l3kk.labs.kodekloud.com", referrer: "https://dkybeqsaea46l3kk.labs.kodekloud.com/"
10.244.0.1 - - [22/Aug/2025:12:02:31 +0000] "GET / HTTP/1.1" 403 555 "https://dkybeqsaea46l3kk.labs.kodekloud.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36"
10.244.0.1 - - [22/Aug/2025:12:03:05 +0000] "GET / HTTP/1.1" 403 555 "https://dkybeqsaea46l3kk.labs.kodekloud.com/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36"
2025/08/22 12:03:05 [error] 80#80: *2 directory index of "/var/www/html/" is forbidden, client: 10.244.0.1, server: _, request: "GET / HTTP/1.1", host: "30008-port-dkybeqsaea46l3kk.labs.kodekloud.com", referrer: "https://dkybeqsaea46l3kk.labs.kodekloud.com/"
^C

```

```bash

thor@jumphost ~$ k cp /home/thor/index.php nginx-phpfpm:/var/www/html -c nginx-container

```

Ensure the php login page by clicking on the website button.

