# Kubernetes Setup and Administration on AWS

This guide walks through installing Kubernetes (via Amazon EKS), configuring clusters, managing resources, and performing essential admin/developer tasks.

---

## 🚀 Prerequisites

- AWS CLI configured (`aws configure`)
- `kubectl` installed
- `eksctl` installed (for cluster provisioning)
- IAM permissions to create EKS resources
- Optional: `helm` for package management

---

## 🔧 Step 1: Install Kubernetes Engine (Amazon EKS)

### 1. Install `eksctl`

```bash
curl --silent --location "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

### 2. Create an EKS Cluster

```bash
eksctl create cluster \
  --name dev-cluster \
  --region ap-south-1 \
  --nodegroup-name dev-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

This creates a fully managed EKS cluster with autoscaling.

---

## 🔐 Step 2: Configure `kubectl` Login

### 1. Update kubeconfig

```bash
aws eks --region ap-south-1 update-kubeconfig --name dev-cluster
```

### 2. Verify Cluster Access

```bash
kubectl get nodes
```

---

## 🧱 Step 3: Create Resources

### 1. Create a Namespace

```bash
kubectl create namespace dev-namespace
```

### 2. Create a Deployment

```bash
kubectl create deployment nginx-deployment \
  --image=nginx \
  --replicas=3 \
  --namespace=dev-namespace
```

### 3. Create a Pod (manually)

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
  namespace: dev-namespace
spec:
  containers:
  - name: sample-container
    image: busybox
    command: ["sleep", "3600"]
```

```bash
kubectl apply -f pod.yaml
```

### 4. Create a Service

```bash
kubectl expose deployment nginx-deployment \
  --type=LoadBalancer \
  --port=80 \
  --namespace=dev-namespace
```

---

## 📋 Step 4: List Resources

```bash
kubectl get all --namespace=dev-namespace
kubectl get pods -o wide
kubectl get services
kubectl get deployments
kubectl get namespaces
```

---

## 🛠️ Step 5: Debugging

### 1. Describe Resources

```bash
kubectl describe pod <pod-name> --namespace=dev-namespace
kubectl describe deployment <deployment-name>
kubectl describe service <service-name>
```

### 2. Check Events

```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
```

### 3. Exec into a Pod

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

---

## 📜 Step 6: Check Logs

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # For crashed pods
kubectl logs -f <pod-name>          # Follow logs
```

---

## 🧹 Step 7: Cleanup

```bash
kubectl delete namespace dev-namespace
eksctl delete cluster --name dev-cluster --region ap-south-1
```

---

## 🧠 Additional Tips for Admins/Developers

- Use `kubectl config view` to inspect kubeconfig
- Use `kubectl top pods` and `kubectl top nodes` for resource metrics (requires Metrics Server)
- Use `helm` for managing complex applications
- Use `kubectl rollout status deployment/<name>` to monitor deployment progress
- Use `kubectl scale deployment <name> --replicas=N` to adjust replicas
- Use `kubectl set image deployment/<name> <container>=<new-image>` for rolling updates

---

## 📚 References

- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [eksctl GitHub](https://github.com/eksctl-io/eksctl)

---
