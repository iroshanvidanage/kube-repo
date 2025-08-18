# Rolling Updates in Kubernetes

Rolling updates in Kubernetes are a powerful way to update your application with zero downtime. They gradually replace old Pods with new ones, ensuring availability throughout the process. Here's a complete breakdown tailored for someone like you, Iroshan, who values reproducibility and precision.

---

## 🔁 Steps for Rolling Updates in Kubernetes

1. **Define a Deployment**  
   Use a Deployment resource to manage your application. It handles rolling updates automatically.

2. **Apply the Initial Deployment**  
   Create and apply the manifest with the initial image/version.

3. **Update the Deployment**  
   Modify the image tag or other spec fields in the manifest or via CLI.

4. **Monitor the Update Progress**  
   Use CLI commands to watch rollout status and ensure success.

5. **Verify the Update**  
   Confirm that new Pods are running and old ones are terminated.

6. **Rollback if Needed**  
   If something goes wrong, you can revert to the previous version.

---

## 📄 Sample Deployment Manifest (`nginx-deployment.yaml`)

- [manifest file](./nginx-deployment.yaml)

---

## 🧪 CLI Commands for Rolling Update Workflow

| Step | Command | Description |
|------|---------|-------------|
| Apply initial deployment | `kubectl apply -f rolling-deployment.yaml` | Deploys version 1.21.0 |
| Get all deployments | `kubectl get deployments` | List deployments |
| Get initial deployment | `kubectl get deployment nginx-deployment` | Get deployment nginx-deployment |
| Describe deployment | `kubectl describe deployment nginx-deployment` | Describe nginx-deployment |
| Update image version | `kubectl set image deployment/nginx-deployment nginx=nginx:1.23.0` | Triggers rolling update |
| Check rollout status | `kubectl rollout status deployment/nginx-deployment` | Monitors progress |
| View history | `kubectl rollout history deployment/nginx-deployment` | Shows previous revisions |
| Rollback if needed | `kubectl rollout undo deployment/nginx-deployment` | Reverts to last stable version |
| Verify Pods | `kubectl get pods -l app=nginx` | Confirms new Pods are running |

---

## ✅ Verification Tips

- Ensure all Pods are in `Running` state: `kubectl describe pod <pod-name>`
- Check logs: `kubectl logs <pod-name>`
- Check image version inside pods:
    - `kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.containers[*].image}'`
- Validate service connectivity if exposed via a Service.


---

---

## Resolve Pod Deployment Issue - Kube Task

```bash

thor@jumphost ~$ k get pods
NAME        READY   STATUS             RESTARTS   AGE
webserver   1/2     ImagePullBackOff   0          32s


thor@jumphost ~$ k describe pod webserver
...

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  76s                default-scheduler  Successfully assigned default/webserver to kodekloud-control-plane
  Normal   Pulling    74s                kubelet            Pulling image "ubuntu:latest"
  Normal   Pulled     72s                kubelet            Successfully pulled image "ubuntu:latest" in 2.870862238s (2.870923985s including waiting)
  Normal   Created    71s                kubelet            Created container sidecar-container
  Normal   Started    71s                kubelet            Started container sidecar-container
  Normal   Pulling    25s (x3 over 75s)  kubelet            Pulling image "httpd:latests"
  Warning  Failed     25s (x3 over 74s)  kubelet            Failed to pull image "httpd:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/httpd:latests": failed to resolve reference "docker.io/library/httpd:latests": docker.io/library/httpd:latests: not found
  Warning  Failed     25s (x3 over 74s)  kubelet            Error: ErrImagePull
  Normal   BackOff    14s (x4 over 71s)  kubelet            Back-off pulling image "httpd:latests"
  Warning  Failed     14s (x4 over 71s)  kubelet            Error: ImagePullBackOff

```

Option 1:
=========

Directly change the image of the container.

`kubectl set image <resource_type>/<resource_name> <container_name>=<image_name>:<image_tag>`

```bash
k set image pod/webserver httpd-container=httpd:latest
```

Option 2:
=========
Generate the YAML file.

```bash
k get pods -o yaml > pod.yaml
```

Edit the spec file: `latests` --> `latest`

```yaml
containerStatuses:
- image: httpd:latests
```

Delete the resource and apply the edited file.

```bash
k delete -f pod.yaml
k create -f pod.yaml
```

```bash
thor@jumphost ~$ k get pod webserver
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          7m37s
```
