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
