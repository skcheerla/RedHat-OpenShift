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

### Monitoring and Observability

Beyond the application probes, you can gain deeper insight into health using OpenShift's built-in monitoring stack:

* **Prometheus and Grafana:** OpenShift is pre-configured with a **Prometheus**-based monitoring stack for collecting cluster and application metrics. You can **enable monitoring for user-defined projects** to collect application-level metrics, and use **Grafana** to visualize them through integrated dashboards.
* **Application Metrics:** Configure your application to expose custom metrics (often via a `/metrics` endpoint using a library like Micrometer or Spring Boot Actuator) that Prometheus can scrape and use for advanced health alerts.

The video below offers an OpenShift-specific tutorial on how to set up these essential health probes. [Monitoring Application Health, Readiness & Liveness Probe (OpenShift Tutorial Part-10)](https://www.youtube.com/watch?v=xGWUypNSTd4) is relevant because it is a direct tutorial demonstrating the configuration of liveness and readiness probes in an OpenShift environment.
http://googleusercontent.com/youtube_content/0
