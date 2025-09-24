Networking in Red Hat OpenShift is a critical component that allows applications to communicate with each other and with external services. It's built upon Kubernetes networking concepts but extends them with additional features and tools. OpenShift networking handles internal pod-to-pod communication, service discovery, and external access to applications.

## Services

A **Service** is an abstract way to expose an application running on a set of Pods as a network service. It provides a single, stable DNS name and IP address for accessing a group of pods, regardless of which node they're on or if pods are replaced or scaled. This makes the application discoverable and accessible. The importance of services lies in providing **service discovery** and **load balancing**. Instead of having to track individual pod IP addresses, which are dynamic and ephemeral, a service provides a persistent endpoint. This allows other pods and external clients to connect to the application without needing to know the specific pod details. Services also distribute network traffic across the pods that back them, acting as a simple load balancer.

## Routes

A **Route** is an OpenShift-specific resource that exposes a service at a host name, like a public DNS record. It's the primary way to expose a service to external clients. Routes provide a public URL and a set of rules for how to route external HTTP and HTTPS traffic to one or more internal services. Routes are important because they enable **external access** and **traffic management**. They provide a secure and manageable way to make your application available on the internet. Routes also handle SSL/TLS termination, allowing you to secure your application's communication with clients.

### TLS Modes for Routes

Routes in OpenShift support different TLS termination modes to handle secure communication.

* **edge** (default): TLS termination happens at the HAProxy edge router. The connection from the router to the pods is unencrypted. This is the simplest and most common mode, as it offloads the encryption overhead from the application pods.
* **passthrough**: The router passes the encrypted traffic directly to the service pods without decrypting it. The pods themselves are responsible for handling TLS termination. This is useful when you want end-to-end encryption or if the application needs to handle client certificates.
* **re-encrypt**: The router terminates the TLS connection from the client, decrypts the traffic, and then re-encrypts it before sending it to the service pods. This ensures a secure connection from the client to the router and from the router to the application pods.

## HAProxy

**HAProxy** is a high-performance, open-source load balancer and reverse proxy that's a key component of OpenShift's routing layer. In OpenShift, the Ingress Controller (the router) often uses HAProxy to manage network traffic. HAProxy handles the routing of external requests to the correct internal services based on the rules defined in Routes. It performs load balancing, health checks, and TLS termination for edge and re-encrypt modes.

## Kube-Proxy

**Kube-Proxy** is a network proxy that runs on each node in a Kubernetes cluster (including OpenShift). It's responsible for implementing the Kubernetes Service concept. Kube-proxy watches the Kubernetes API server for changes to services and endpoints. When a new service is created, Kube-proxy configures the networking rules on the node (using iptables or IPVS) to forward traffic destined for the service's cluster IP to the correct backing pods. It acts as a rudimentary load balancer for service-level traffic within the cluster.

---

## Service Commands

Here are some common `oc` commands for managing services:

* `oc get svc` : Lists all services in the current project.
* `oc create service clusterip <name> --tcp=<port>:<target-port>` : Creates a new ClusterIP service.
* `oc expose deployment <deployment-name> --port=<port> --target-port=<target-port>` : Creates a service that exposes a deployment.
* `oc describe svc <service-name>` : Shows detailed information about a service, including its IP, ports, and selectors.
* `oc delete svc <service-name>` : Deletes a service.

---

## Route Commands

Here are some common `oc` commands for configuring routes:

* `oc get route` : Lists all routes in the current project.
* `oc create route edge --service=<service-name> --insecure-policy=Redirect` : Creates a new edge route with a redirect policy.
* `oc create route passthrough --service=<service-name>` : Creates a new passthrough route.
* `oc create route reencrypt --service=<service-name> --dest-ca-cert=<path/to/ca.crt>` : Creates a new re-encrypt route.
* `oc set tls route <route-name> --cert=<path/to/cert.pem> --key=<path/to/key.pem> --ca-cert=<path/to/ca.pem> --insecure-policy=Redirect` : Adds a TLS certificate to an existing route.
* `oc describe route <route-name>` : Provides detailed information about a route.
* `oc delete route <route-name>` : Deletes a route.

In OpenShift, the built-in **HAProxy** router supports three primary modes for TLS termination, which you can configure on a per-route basis. These modes determine where the secure TLS connection is decrypted.

### Edge Termination
This is the most common and simplest termination mode. In this mode, the TLS connection is terminated at the OpenShift router (HAProxy). The router decrypts the incoming HTTPS traffic and then forwards the unencrypted (plain HTTP) traffic to the application's pod.

* **Pros:** The application itself doesn't need to handle certificates or TLS encryption, which reduces the computational overhead on the application. The router manages all certificates, making it a centralized and efficient solution.
* **Cons:** The connection between the router and the application is not encrypted. While this is typically acceptable in a trusted, internal OpenShift network, it may not meet strict security compliance requirements that demand end-to-end encryption.
* **Use Case:** Ideal for applications that are not natively TLS-aware or for environments where internal network traffic is considered secure.

### Re-encryption Termination
Re-encryption termination provides end-to-end encryption. The router first terminates the client's TLS connection (just like with edge termination). However, instead of forwarding unencrypted traffic, it establishes a new, secure TLS connection to the application's pod and re-encrypts the traffic. The application pod must be configured to handle the TLS connection.

* **Pros:** Provides end-to-end encryption from the client to the application pod, satisfying stringent security requirements.
* **Cons:** Requires more configuration and resource overhead, as both the router and the application must handle TLS encryption and decryption.
* **Use Case:** Critical for applications handling sensitive data, or in environments with a zero-trust security model where all traffic, even internal, must be encrypted.

### Passthrough Termination
In passthrough termination, the OpenShift router doesn't terminate the TLS connection at all. It simply passes the encrypted traffic directly to the application's pod. The application is entirely responsible for managing the TLS certificate and decrypting the traffic. The router only uses the **Server Name Indication (SNI)** header to route the traffic to the correct service.

* **Pros:** The application maintains full control over its TLS configuration. This is the most secure option for end-to-end encryption because the traffic is never decrypted by an intermediary.
* **Cons:** Since the router cannot inspect the traffic's HTTP headers, it's unable to perform any Layer 7 routing or load balancing based on HTTP paths or cookies. All routing decisions must be made at Layer 4 (TCP) using SNI.
* **Use Case:** Necessary for applications that require mutual TLS (mTLS) or when the application needs to handle the certificate directly for custom authentication or a specific protocol.



<img width="948" height="478" alt="image" src="https://github.com/user-attachments/assets/f229e5c6-c277-4d9b-afa7-d8142bed2abf" />
