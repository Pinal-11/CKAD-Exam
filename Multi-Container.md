# Multi-Container Pods

A **Multi-Container Pod** is a Kubernetes Pod that contains more than one container.

For example, a Pod can contain:

1. `app-container`
2. `main-container`

Containers within the same Pod:

* Share the **same lifecycle** — they are created and destroyed together.
* Share the **same network namespace**, so containers can communicate with each other using `localhost`.
* Can share the **same volumes**.
* Are scheduled together on the **same Kubernetes node**.

---

# Multi-Container Pod Design Patterns

There are three common design patterns for multi-container Pods:

1. Co-Located Containers
2. Init Containers
3. Sidecar Containers

---

## 1. Co-Located Containers

In the **Co-Located Containers** pattern, multiple containers run together in the same Pod and are generally required for the application to function.

Both containers run simultaneously and may depend on each other.

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080

    - name: main-app
      image: main-app
```

### Key Points

* Both containers run at the same time.
* They share the Pod's network namespace.
* They can communicate using `localhost`.
* They can share volumes.
* They have the same Pod lifecycle.

---

## 2. Init Containers

An **Init Container** is a container that runs **before the main application containers start**.

Init containers are useful for performing initialization tasks such as:

* Checking whether a database is available.
* Waiting for another service to become available.
* Downloading or preparing configuration.
* Performing setup or initialization tasks.

The main application container starts **only after all Init Containers have completed successfully**.

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  initContainers:
    - name: db-checker
      image: busybox
      command: ["sh", "-c", "wait-for-db-to-start.sh"]

    - name: api-checker
      image: busybox
      command: ["sh", "-c", "wait-for-another-api-checker.sh"]

  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080
```

### Execution Flow

```text
db-checker
     ↓
api-checker
     ↓
web-app starts
```

### Key Points

* Init containers run **sequentially**.
* Each Init Container must complete successfully before the next one starts.
* The application containers start only after all Init Containers finish.
* Init Containers normally run to completion and then stop.

---

## 3. Sidecar Container

A **Sidecar Container** is a container that runs alongside the main application container and provides an additional supporting function.

Unlike a traditional Init Container, a Sidecar Container **continues running while the main application is running**.

Common Sidecar use cases include:

* Log collection and shipping.
* Monitoring.
* Proxying.
* Configuration synchronization.
* Security or networking support.

### Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
    - name: web-app
      image: web-app
      ports:
        - containerPort: 8080

  initContainers:
    - name: log-shipper
      image: busybox
      restartPolicy: Always
      command: ["sh", "-c", "setup-log-shipper.sh"]
```

> **Note:** Modern Kubernetes supports native Sidecar Containers by defining a container under `initContainers` with `restartPolicy: Always`. This allows the sidecar to start during Pod initialization and continue running alongside the main application container.

### Execution Flow

```text
Sidecar starts
      ↓
Main container starts
      ↓
Both containers run
      ↓
Main container stops
      ↓
Sidecar terminates according to Pod lifecycle
```

---

# Quick Comparison

| Pattern                   | When Does It Run?              | Continues Running? | Main Purpose                                    |
| ------------------------- | ------------------------------ | ------------------ | ----------------------------------------------- |
| **Co-Located Containers** | Together with other containers | Yes                | Multiple tightly coupled application components |
| **Init Container**        | Before main containers         | No                 | Initialization and setup                        |
| **Sidecar Container**     | During Pod startup             | Yes                | Supporting the main application                 |

## Easy Way to Remember

```text
Co-Located
    → Multiple containers working together

Init Container
    → Prepare something BEFORE the app starts

Sidecar Container
    → Support the app WHILE it is running
```
