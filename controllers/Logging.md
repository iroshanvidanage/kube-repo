# Execute commands in the container

Executing commands inside containers, and inspecting custom log files within pods.

---

## 🧠 Part 1: Executing Commands in Containers

To run commands inside a container of a pod, use `kubectl exec`.

```bash
kubectl exec -it <pod-name> -c <container-name> -- <command>
```

### 🔧 Examples:
- **Open a shell inside a container**:
  ```bash
  kubectl exec -it my-pod -c my-container -- /bin/bash
  ```
  Or use `/bin/sh` if bash isn’t available.

- **Run a one-off command**:
  ```bash
  kubectl exec my-pod -c my-container -- ls /app/logs
  ```

> [!TIP]💡
> If your pod has only one container, you can omit `-c <container-name>`.

---

## 📁 Part 2: Checking Custom Log Files in Pods/Containers

Kubernetes logs via `kubectl logs` only show stdout/stderr. For custom log files (e.g., `/var/log/myapp.log`), you’ll need to `exec` into the container and inspect them manually.

### 🔍 Steps:
1. **Exec into the container**:
   ```bash
   kubectl exec -it my-pod -- /bin/sh
   ```

2. **Navigate to the log file location**:
   ```bash
   cd /var/log
   ```

3. **View the log file**:
   ```bash
   cat myapp.log
   ```
   Or use:
   ```bash
   tail -n 100 myapp.log
   less myapp.log
   ```

### 🛡️ Bonus: Make logs accessible via stdout
If you want `kubectl logs` to pick up custom logs, you can symlink or redirect them to stdout:

```bash
ln -sf /var/log/myapp.log /proc/1/fd/1
```

Or modify your app to log directly to stdout/stderr.

```py
# !/bin/python
import logging
import sys

logging.basicConfig(stream=sys.stdout, level=logging.INFO)
logger = logging.getLogger()

logger.info("Info to stdout")
logger.error("Error to stderr")

```
---

## 🧪 Verification Tips
- Use `kubectl describe pod <pod-name>` to confirm container names and volume mounts.
- If your logs are in ephemeral storage, make sure your container has enough space and retention.
- Consider mounting a persistent volume if logs need to survive pod restarts.

---

```sh
kubectl exec -n datacenter time-check -- tail -f /opt/dba/time/time-check.log
```

