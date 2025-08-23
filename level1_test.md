# Kubernetes Test Level 1: KodeKloud

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-nautilus-t1q5
  labels:
    app.kubernetes.io/name: dev-nginx
spec:
  containers:
  - name: red-main-nautilus-t1q5
    image: debian:latest
    command: ['/bin/bash', '-c', 'sleep 1000']
  initContainers:
  - name: red-init-nautilus-t1q5
    image: debian:latest
    command: ['/bin/bash', '-c', 'echo "Welcome!"']
```

---

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset-t3q4
  labels:
    app: nginx_app_t3q4
    type: front-end-t3q4
spec:
  replicas: 4
  selector:
    matchLabels:
      type: front-end-t3q4
  template:
    metadata:
      labels:
        type: front-end-t3q4
    spec:
      containers:
      - name: nginx-container-t3q4
        image: nginx:latest
        ports:
          - containerPort: 80
```

---

```yaml
apiVersion: batch/v1V
kind: Job
metadata:
  name: countdown-nautilus-t3q2
spec:
  template:
    metadata:
      name: countdown-nautilus-t3q2
    spec:
      containers:
      - name: container-countdown-nautilus-t3q2
        image: debian:latest
        command: ["sleep",  "5"]
      restartPolicy: Never
```

---

```sh
  ----     ------     ----                  ----               -------
  Normal   Scheduled  22m                   default-scheduler  Successfully assigned default/python-deployment-nautilus-t4q4-5db9686879-rs784 to kodekloud-control-plane
  Normal   Pulling    21m (x4 over 22m)     kubelet            Pulling image "poroko/flask-app-demo"
  Warning  Failed     21m (x4 over 22m)     kubelet            Failed to pull image "poroko/flask-app-demo": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/poroko/flask-app-demo:latest": failed to resolve reference "docker.io/poroko/flask-app-demo:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     21m (x4 over 22m)     kubelet            Error: ErrImagePull
  Warning  Failed     20m (x6 over 22m)     kubelet            Error: ImagePullBackOff
  Normal   BackOff    2m22s (x86 over 22m)  kubelet            Back-off pulling image "poroko/flask-app-demo"

```
