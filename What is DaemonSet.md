A **DaemonSet** in a Red Hat OpenShift cluster (which is based on Kubernetes) is a workload resource that ensures a **copy of a specific Pod runs on all (or a subset of) eligible nodes** in the cluster.

Its primary purpose is to deploy system-level services or agents that need to be present and running on every node for cluster-wide functions.

***

## Key Characteristics and Functions

* **One Pod Per Node:** The fundamental concept of a DaemonSet is that it creates exactly one replica of a defined Pod on each node that matches its specifications (unless you use node selectors or affinity rules to target a subset of nodes).
* **Automatic Scaling with Nodes:**
    * When a new node joins the cluster, the DaemonSet automatically adds a Pod replica to that node.
    * When a node is removed from the cluster, the DaemonSet automatically performs garbage collection to remove the associated Pod.
* **Use Cases:** DaemonSets are ideal for deploying background, long-running services that need to run continuously on every node for system health and functionality. Common examples include:
    * **Log Collectors:** Agents like Fluentd or Logstash to collect logs from each node and forward them to a central logging system.
    * **Monitoring Agents:** Agents like Prometheus Node Exporter or Datadog agent to collect metrics and monitor the health and performance of the node itself.
    * **Cluster Storage Daemons:** Daemons for distributed storage systems like Ceph or Gluster.
    * **Network Plugins:** Components like `kube-proxy` that ensure proper networking across all nodes.

## DaemonSet vs. Deployment

While both manage Pods, their goals are different:

| Feature | DaemonSet | Deployment |
| :--- | :--- | :--- |
| **Scaling Logic** | One Pod per node (or subset) | Scales by the desired number of Pod replicas, spread across multiple nodes. |
| **Purpose** | Node-level services requiring full node coverage (e.g., monitoring, logging). | Stateless application workloads (e.g., web frontends, REST APIs). |
| **New Nodes** | Automatically deploys a Pod to the new node. | Does **not** automatically deploy new Pods; depends on the existing replica count. |

***

You can find more details and an example of its use in OpenShift in this video: [OpenShift - and Kubernetes Daemonsets](https://www.youtube.com/watch?v=_nuv59uFBMM). This video explains and demonstrates how to use DaemonSets within an OpenShift environment.
http://googleusercontent.com/youtube_content/0
