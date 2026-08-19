# Kubernetes Readiness and Liveness Probes

Kubernetes uses **probes** to determine the health and availability of containers.

The two most commonly used probes are:

1. **Readiness Probe**
2. **Liveness Probe**

---

# 1. Pod Lifecycle and Status

Before understanding probes, it is important to understand the basic Pod lifecycle.

## Pod Status

### Pending

The Pod has been accepted by Kubernetes but has not yet started running.

For example:

* The Pod has not been scheduled to a node yet.
* The scheduler is determining which node should run the Pod.
* The container image may still need to be pulled.

```text
Pod Created
     ↓
Pending
     ↓
Scheduler selects a Node
```

### Container Creating

Once the scheduler selects a node:

* The Pod is assigned to that node.
* The kubelet on that node starts preparing the Pod.
* Container images are pulled if they are not already available.
* Containers are created.

```text
Pending
   ↓
Scheduled
   ↓
Container Creating
   ↓
Container Started
```

> **Important:** `Container Creating` is commonly seen as a container state/reason in commands such as `kubectl describe pod`; it is not one of the official high-level Pod phase values.

### Running

The Pod enters the **Running** phase when the Pod has been bound to a node and at least one of its containers has started.

However, **Running does not necessarily mean that the application is ready to receive traffic**.

This is where the **Readiness Probe** becomes important.

---

# 2. Pod Conditions

In addition to the Pod phase, Kubernetes maintains several **Pod Conditions**.

Common conditions include:

| Condition         | Meaning                                          |
| ----------------- | ------------------------------------------------ |
| `PodScheduled`    | Pod has been successfully scheduled to a node.   |
| `Initialized`     | All Init Containers have completed successfully. |
| `ContainersReady` | All containers in the Pod are ready.             |
| `Ready`           | The Pod is ready to receive traffic.             |

> **Note:** The condition is `ContainersReady` (plural), not `ContainerReady`.

Example:

```text
PodScheduled
     ↓
Initialized
     ↓
ContainersReady
     ↓
Ready
```

---

# 3. Readiness Probe

## Definition

A **Readiness Probe** determines whether a container is **ready to receive network traffic**.

If the readiness probe fails, Kubernetes considers the container **not ready**.

The container is **not restarted** because of a readiness probe failure.

Instead, Kubernetes removes the Pod from the endpoints of Services that select that Pod, so normal Service traffic is no longer sent to it.

## Use Case

Readiness probes are useful when:

* An application takes time to initialize.
* The application needs to establish a database connection before serving requests.
* The application needs to load configuration.
* A temporary dependency is unavailable.
* The application is running but temporarily unable to serve requests.

### Example

Suppose an application takes 30 seconds to initialize.

Without a readiness probe:

```text
Pod starts
   ↓
Container Running
   ↓
Traffic immediately sent to application
   ↓
Application is still initializing
   ↓
Requests may fail
```

With a readiness probe:

```text
Pod starts
   ↓
Container Running
   ↓
Readiness Probe fails
   ↓
Pod is NOT added to Service endpoints
   ↓
Application becomes ready
   ↓
Readiness Probe succeeds
   ↓
Pod receives traffic
```

---

# 4. Liveness Probe

## Definition

A **Liveness Probe** determines whether a container is still functioning correctly.

If the liveness probe fails repeatedly according to its configured thresholds, the kubelet **restarts the container**.

## Use Case

Liveness probes are useful when an application can become stuck or deadlocked while the container itself is still running.

For example:

```text
Container is Running
        ↓
Application becomes deadlocked
        ↓
Container process still exists
        ↓
Liveness Probe fails
        ↓
Kubelet restarts the container
        ↓
Application recovers
```

### Important Difference

A liveness probe answers:

> **"Is my application still alive?"**

A readiness probe answers:

> **"Can my application receive traffic right now?"**

---

# 5. Readiness vs Liveness

| Feature                            | Readiness Probe                                | Liveness Probe                                  |
| ---------------------------------- | ---------------------------------------------- | ----------------------------------------------- |
| Purpose                            | Determines whether the app can receive traffic | Determines whether the app is still functioning |
| Probe failure                      | Pod becomes NotReady                           | Container may be restarted                      |
| Removes Pod from Service endpoints | Yes                                            | No                                              |
| Restarts container                 | No                                             | Yes                                             |
| Useful during startup              | Yes                                            | Yes, but carefully configured                   |
| Handles temporary unavailability   | Yes                                            | Usually no                                      |
| Handles deadlocked application     | Not primarily                                  | Yes                                             |

### Easy Way to Remember

```text
Readiness → "Should I send traffic to this Pod?"

Liveness  → "Should I restart this container?"
```

---

# 6. Types of Probes

Kubernetes supports several probe mechanisms.

## HTTP Probe

Kubernetes sends an HTTP GET request to the specified endpoint.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
```

If the HTTP endpoint returns a successful response, the probe succeeds.

### Use Case

Use HTTP probes when your application exposes a health endpoint such as:

```text
/api/health
/api/ready
/healthz
```

---

## TCP Probe

Kubernetes attempts to establish a TCP connection to the specified port.

Example:

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

If the TCP connection succeeds, the probe succeeds.

### Use Case

TCP probes are useful when the application does not expose an HTTP health endpoint but listens on a TCP port.

---

## Exec Probe

Kubernetes executes a command inside the container.

Example:

```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

If the command exits with status code `0`, the probe succeeds.

### Use Case

Exec probes are useful when application health can be determined by running a command or checking a file.

---

# 7. Probe Configuration Parameters

## `initialDelaySeconds`

Specifies how many seconds Kubernetes should wait before performing the first probe.

```yaml
initialDelaySeconds: 10
```

Example:

If your application normally takes 10 seconds to start:

```text
Container starts
      ↓
Wait 10 seconds
      ↓
First probe
```

---

## `periodSeconds`

Specifies how frequently Kubernetes performs the probe.

```yaml
periodSeconds: 5
```

This means Kubernetes performs the probe approximately every 5 seconds.

```text
Probe → 5s → Probe → 5s → Probe → 5s → Probe
```

---

## `failureThreshold`

Specifies how many consecutive probe failures are required before Kubernetes considers the probe failed.

```yaml
failureThreshold: 8
```

The default value is **3**.

For example, with:

```yaml
failureThreshold: 8
periodSeconds: 5
```

Kubernetes needs 8 consecutive failures before taking the corresponding action.

> **Important:** The exact time before an action occurs can also depend on probe startup behavior and other configured parameters. Do not simply assume it is always `failureThreshold × periodSeconds`.

---

## `successThreshold`

Specifies the number of consecutive successful probes required to consider a previously failing probe successful.

For most probes, the default is `1`.

Example:

```yaml
successThreshold: 1
```

---

## `timeoutSeconds`

Specifies how long Kubernetes waits for a probe to complete.

```yaml
timeoutSeconds: 2
```

If the probe does not complete within the timeout, it is considered a failure.

---

# 8. Complete Example

A Pod can use both readiness and liveness probes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080

      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 5
        failureThreshold: 8

      livenessProbe:
        httpGet:
          path: /api/health
          port: 8080
        initialDelaySeconds: 30
        periodSeconds: 10
        failureThreshold: 3
```

### What Happens?

```text
Container starts
       ↓
Readiness Probe
       ↓
Not Ready
       ↓
Application finishes initialization
       ↓
Readiness Probe succeeds
       ↓
Pod receives Service traffic
       ↓
Application continues running
       ↓
Application becomes unhealthy
       ↓
Liveness Probe fails repeatedly
       ↓
Kubelet restarts container
```

---

# 9. Using HTTP, TCP, and Exec

Only **one probe handler** should be configured for a particular probe.

### HTTP

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
```

### TCP

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

### Exec

```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

You should **not** define three separate `readinessProbe` sections in the same container expecting Kubernetes to run all three. Configure one probe handler per probe.

---

# 10. Startup Probe

Kubernetes also provides a **Startup Probe**, which is especially useful for applications that take a long time to start.

## Definition

A Startup Probe determines whether the application inside a container has successfully started.

While the startup probe is running successfully:

* Kubernetes does not run the liveness or readiness probes.
* This prevents a slow-starting application from being unnecessarily restarted by the liveness probe.

Example:

```yaml
startupProbe:
  httpGet:
    path: /api/health
    port: 8080
  failureThreshold: 30
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /api/health
    port: 8080
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  periodSeconds: 5
```

### Probe Flow

```text
Container starts
      ↓
Startup Probe
      ↓
Application successfully starts
      ↓
Startup Probe succeeds
      ↓
Readiness + Liveness probes begin
      ↓
Readiness → Controls traffic
Liveness  → Controls restarts
```

---

# 11. Kubernetes Exam Points

Remember these key points:

* **Readiness Probe** → Determines whether the Pod should receive traffic.
* **Liveness Probe** → Determines whether the container should be restarted.
* **Startup Probe** → Determines whether a slow-starting application has started.
* Readiness failure **does not restart** the container.
* Liveness failure can cause the kubelet to **restart the container**.
* Readiness failure causes the Pod to become **NotReady** and removes it from Service endpoints.
* `httpGet` → HTTP health check.
* `tcpSocket` → TCP connection check.
* `exec` → Executes a command inside the container.
* `initialDelaySeconds` → Delay before the first probe.
* `periodSeconds` → Frequency of probes.
* `failureThreshold` → Consecutive failures required before failure is acted upon.
* `successThreshold` → Consecutive successes required to recover.
* `timeoutSeconds` → Maximum time allowed for a probe.
* A Pod being **Running does not necessarily mean it is Ready**.

## One-Line Memory Trick

```text
Startup  → "Has the application started?"
Readiness → "Can the application receive traffic?"
Liveness  → "Is the application still alive?"
```
