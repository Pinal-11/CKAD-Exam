# Logging

## Docker Logs

To view logs from a Docker container:

```bash
docker logs <container-name>
```

To follow live logs:

```bash
docker logs -f <container-name>
```

> `-f` means **follow** the logs and continuously display new log entries.

---

## Kubernetes Logs

To view logs from a Pod:

```bash
kubectl logs <pod-name>
```

To follow live logs:

```bash
kubectl logs -f <pod-name>
```

### Multiple Containers in a Pod

If a Pod contains multiple containers, specify the container name:

```bash
kubectl logs -f <pod-name> -c <container-name>
```

Example:

```bash
kubectl logs -f my-pod -c web-app
```

> `-c` specifies the **container** whose logs you want to view.

### Quick Memory

```text
Docker:
docker logs -f <container>

Kubernetes:
kubectl logs -f <pod>

Multiple containers:
kubectl logs -f <pod> -c <container>
```

**Important:** The correct command is `kubectl logs` (plural), not `kubectl log`.
