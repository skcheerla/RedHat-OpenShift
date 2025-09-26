The primary method for performing application health checks in a Red Hat OpenShift cluster is by configuring **Kubernetes health probes** for your application's containers. OpenShift, being based on Kubernetes, utilizes three main types of probes:

## 1. Liveness Probes

**Liveness probes** determine if a container is still running and in a healthy state. If a liveness probe fails, OpenShift (specifically the kubelet) considers the container "dead" and automatically **restarts** it according to the pod's restart policy. This is useful for catching issues like:

* **Deadlocks:** Where an application process is running but unable to make progress.
* **Unrecoverable internal states:** Issues like a memory leak or a fatal application error that only a restart can fix.

## 2. Readiness Probes

**Readiness probes** determine if a container is ready to start servicing network requests. If a readiness probe fails, OpenShift **removes the pod's IP address from the service endpoints**. This ensures that the application's built-in load balancer (the OpenShift Router/Ingress Controller) **stops sending traffic** to that specific pod instance. This is essential for:

* **During startup:** Preventing traffic from reaching an application that is still initializing, loading data, or connecting to databases.
* **Graceful shutdowns/updates:** Ensuring a container is only taken out of rotation when it fails the readiness check, but not necessarily restarted (unless the liveness probe fails).

## 3. Startup Probes

**Startup probes** were introduced to handle applications that can take a long time to start up. If a startup probe is configured, it **disables liveness and readiness checks** until it succeeds. If the startup probe fails, the container is killed and subject to the pod's restart policy. This is important for:

* **Slow-starting applications:** Preventing the liveness probe from prematurely restarting a container that is simply taking a while to initialize.

***

### Configuring Probes

You can configure these probes in your application's Deployment Configuration (`DeploymentConfig`) or Deployment (`Deployment`) using the OpenShift Web Console or the command line interface (`oc`).

#### Probe Types

Each probe type can be configured with one of the following methods:

1.  **HTTP GET:** Sends an HTTP request to a specified path and port. A successful response is an HTTP status code between 200 and 399.
2.  **Container Command:** Executes a command inside the container. The probe is successful if the command exits with a status code of 0.
3.  **TCP Socket:** Attempts to establish a TCP connection to the specified port. Success is establishing the connection.

#### Key Configuration Parameters

Common parameters you set for any probe include:

* **`initialDelaySeconds`**: The time delay, in seconds, after the container starts before the probe is first executed.
* **`periodSeconds`**: How often the probe should be performed (default is 10 seconds).
* **`timeoutSeconds`**: The number of seconds after which the probe times out (default is 1 second).
* **`failureThreshold`**: The number of consecutive failures needed for the probe to be considered failed (e.g., resulting in a restart for liveness or traffic removal for readiness).

***


You can configure application health checks in Red Hat OpenShift using **Startup**, **Readiness**, and **Liveness Probes**. These are defined in the Pod specification within your OpenShift resource (like a `Deployment` or `DeploymentConfig`).

Here is a step-by-step guide focusing on the YAML configuration method, which is the most common and comprehensive way to define these probes.

-----

## 1\. Understanding the Probes

Before configuring, it's essential to understand the role of each probe:

| Probe | Purpose | Action on Failure |
| :--- | :--- | :--- |
| **Startup Probe** (`startupProbe`) | Checks if the application **has successfully started**. | Prevents other probes (Liveness/Readiness) from running until it succeeds. If it fails *repeatedly* (past its failure threshold), the pod is killed and restarted. |
| **Readiness Probe** (`readinessProbe`) | Checks if the application is **ready to serve traffic**. | The pod's IP is removed from the service endpoints. Traffic is blocked until the probe succeeds. The pod is **not** restarted. |
| **Liveness Probe** (`livenessProbe`) | Checks if the application is **currently running and healthy**. | The container is killed by the Kubelet, and the pod is subjected to its restart policy (usually restarting the container). |

-----

## 2\. Step-by-Step Configuration using YAML

You'll define the probes within the container specification of your OpenShift resource (e.g., a `Deployment`).

### Step 1: Create or Edit the Deployment YAML

Create a new file (e.g., `myapp-deployment.yaml`) or edit your existing deployment file. The probes are configured under `spec.template.spec.containers[0]`.

### Step 2: Define the Probe Configurations

Choose a probe type (`httpGet`, `exec`, or `tcpSocket`) and define the parameters for each probe.

#### Example YAML Configuration (HTTP GET)

This example uses an **HTTP GET** check, which is common for web applications, assuming your application exposes a health check endpoint at `/health` on port `8080`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: myregistry/my-app:latest
        ports:
        - containerPort: 8080
        
        # --- STARTUP PROBE (Optional, for slow-starting apps) ---
        startupProbe:
          httpGet:
            path: /health  # Endpoint to check
            port: 8080     # Port to check
          initialDelaySeconds: 5  # Wait 5 seconds before first check
          periodSeconds: 10       # Check every 10 seconds
          failureThreshold: 12    # Allow 12 failures (12 * 10s = 120s total startup time)

        # --- READINESS PROBE ---
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15 # Wait 15 seconds after container starts before first check
          periodSeconds: 5        # Check every 5 seconds
          timeoutSeconds: 2       # If the response takes > 2 seconds, consider it a failure
          failureThreshold: 3     # Allow 3 consecutive failures before marking as Not Ready

        # --- LIVENESS PROBE ---
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 45 # Wait 45 seconds after container starts (or startupProbe success)
          periodSeconds: 10       # Check every 10 seconds
          timeoutSeconds: 5       # If the response takes > 5 seconds, consider it a failure
          failureThreshold: 3     # Allow 3 consecutive failures before restarting the container
```

### Step 3: Apply the Configuration

Use the OpenShift CLI (`oc`) to apply the configuration.

```bash
# Log in to OpenShift if you haven't already
oc login ... 

# Switch to your project (namespace)
oc project <your-project-name>

# Apply the Deployment file
oc apply -f myapp-deployment.yaml
```

### Step 4: Verify the Health Checks

You can check the status of your pods and the probe events using `oc describe`.

```bash
# Get the name of a running pod
oc get pods

# Describe the pod to see Probes and Events
oc describe pod <pod-name-from-above> | grep -E 'Liveness|Readiness|Startup|Events:' -A 10
```

In the output, you will see sections for **Liveness Probe**, **Readiness Probe**, and **Startup Probe** detailing their configuration, as well as **Events** showing when a probe failed and the resulting action (e.g., `Liveness probe failed, killing container`).

-----

## 3\. Alternative Probe Types

You can use different probe types depending on your application:

### A. Exec Probe

Runs a command inside the container. It's successful if the command exits with status `0`.

**Example (Liveness Probe - Exec):**
This probe checks if a critical file exists in the container.

```yaml
        livenessProbe:
          exec:
            command:
            - cat
            - /tmp/app-is-alive
          initialDelaySeconds: 20
          periodSeconds: 10
```

### B. TCP Socket Probe

Attempts to open a socket to the container on the specified port. It's successful if the connection is established.

**Example (Readiness Probe - TCP Socket):**
This probe checks if the application is listening on port `5432` (common for PostgreSQL).

```yaml
        readinessProbe:
          tcpSocket:
            port: 5432
          initialDelaySeconds: 30
          periodSeconds: 5
```

-----

## 4\. Quick CLI Configuration (`oc set probe`)

For quickly adding or modifying basic probes without editing the full YAML file, you can use the `oc set probe` command. This is primarily for existing Deployments/DeploymentConfigs.

**Example: Adding a Readiness Probe (HTTP GET)**

```bash
oc set probe deployment/my-app-deployment --readiness \
  --get-url=http://:8080/health \
  --initial-delay-seconds=15 \
  --period-seconds=5
```

**Example: Adding a Liveness Probe (TCP Socket)**

```bash
oc set probe deployment/my-app-deployment --liveness \
  --open-tcp=8080 \
  --initial-delay-seconds=45 \
  --period-seconds=10 \
  --failure-threshold=3
```

### Monitoring and Observability

Beyond the application probes, you can gain deeper insight into health using OpenShift's built-in monitoring stack:

* **Prometheus and Grafana:** OpenShift is pre-configured with a **Prometheus**-based monitoring stack for collecting cluster and application metrics. You can **enable monitoring for user-defined projects** to collect application-level metrics, and use **Grafana** to visualize them through integrated dashboards.
* **Application Metrics:** Configure your application to expose custom metrics (often via a `/metrics` endpoint using a library like Micrometer or Spring Boot Actuator) that Prometheus can scrape and use for advanced health alerts.

The video below offers an OpenShift-specific tutorial on how to set up these essential health probes. [Monitoring Application Health, Readiness & Liveness Probe (OpenShift Tutorial Part-10)](https://www.youtube.com/watch?v=xGWUypNSTd4) is relevant because it is a direct tutorial demonstrating the configuration of liveness and readiness probes in an OpenShift environment.
http://googleusercontent.com/youtube_content/0
