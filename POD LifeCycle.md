oc new-app
client submits higher levelobject

this request goes to like below

Kube API server

 -he will authenticate and check his authorization

he ll check resource quota, scc, image policies,image singature checks , service accont injection
node selector, namespace selector ,default mutations and image pull secrets


persist final pod object to etcd



etcd 

stores the pod object in cluster state
any controller or kubelet reads etcd via apiserver


controllers 

replication controller -- if deployement mentioned replica
deployement controller -- create pod objects wen create a deployement
service account controller --- maintains & secrets fr svc accounts
pvc controller   - handling binding and provisioning operations (pv/pvs)

attach and detach controller for block storage 
openshift controller -- imagestram controllers , route controllers, service controllers

All these controllers hhelp to reconsile respective pods based on the request


even though
Pod is unscheduled

scheduler 

scheduler component comes into picture

watches fr pods with specific node names
runs predicates and scoring - based on node capacity,node taint, affinity
topology constraint, etc and choose the bestnode and writes binding to the pod object via api servers

controlplane 

controlplane updates & all controllers react
deployement controller node pod is scheduled
end point controller will later update service updates
images stream controller reacts if openshift imagestream is called

Node is decided


Kubelet


then each worker node has kublet

kubelet watches api server for pod object assigned to node

validates pod specifications locally security constriants ,cgroups

ensures volumes pvcs are attached and mounted

pulls container images via container run time (crio)

crio

pull images from registry
create containers namespaces ,secrets contexts,cgroups
it start the process inside the container

 

Popular Container Runtimes

Runtime	         Used By	                Notes
runc	       Low-level runtime	     Actually runs containers via Linux
Docker	       Legacy/runtime tool	     Uses runc under the hood
containerd	   Kubernetes, Docker	     Lightweight, CRI-compatible
CRI-O	       OpenShift, Kubernetes   	Built for Kubernetes 



Networking OVN kubernetes

it is equivalent kubeproxy
- allocates pod ips and program node networking
routing rules ip allocation to pod,nodeport
ensures service connectivity and netwkr policy is enforced

Networking integreates with routes/ingress for external access



pod lifecycle

kubelet runs readiness and liveness  probes
when readiness probe succeceeds pods bcmes ready and service endpoints updates
kubelets updates podstatus back to api server


API server will update /etcd with the pod status



SCC

taint  -given to a node
tolerate - tolerate given to a pod

tolerate will take preceedance and stil can run on a tainted node




Sure! Let’s break down **CRI-O** carefully, especially in the context of Red Hat / OpenShift.

---

## 🔹 What is CRI-O?

**CRI-O** is a **lightweight container runtime for Kubernetes**.

It implements the **Kubernetes Container Runtime Interface (CRI)** to run **OCI-compliant containers** (Docker/Podman-style containers) without needing a full Docker engine.

* **CRI** = Container Runtime Interface → Kubernetes uses this interface to communicate with container runtimes.
* **OCI** = Open Container Initiative → Defines standards for container images and runtimes.

In simple terms:

> CRI-O is a minimal runtime that lets Kubernetes run containers directly, following OCI standards, without the overhead of Docker.

---

## 🔹 Why CRI-O in OpenShift/RHEL?

1. **Lightweight** → Only implements what Kubernetes needs.
2. **Secure** → Works well with SELinux, seccomp, and other RHEL security features.
3. **Compatible** → Runs any OCI-compliant container image (like images from Docker Hub).
4. **Supported by Red Hat** → Default container runtime for OpenShift 4.x+ and RHEL 9 Kubernetes.

---

## 🔹 How it Works in OpenShift

**Flow of running a Pod:**

1. **You create a Pod** → YAML goes to **API server**.
2. **Kubelet receives it** → Kubelet uses **CRI** to ask the runtime to start containers.
3. **CRI-O runs the container** → Pulls the image, creates container process, sets up networking, applies cgroups for resource limits.

**Diagram (simplified):**

```
Pod YAML
   ↓
API Server
   ↓
Kubelet
   ↓
CRI-O
   ↓
Linux Kernel (cgroups, namespaces, SELinux)
   ↓
Container process runs
```

---

## 🔹 Commands to interact with CRI-O

* **Check CRI-O version:**

```bash
crio --version
```

* **Manage CRI-O service (RHEL/OpenShift node):**

```bash
systemctl status crio
systemctl start crio
systemctl enable crio
```

* **Logs:**

```bash
journalctl -u crio
```

---

✅ **Summary:**
CRI-O is a **lightweight, Kubernetes-native container runtime** that OpenShift uses instead of Docker to run pods securely and efficiently, while leveraging Linux features like **cgroups** for resource management.

---

If you want, I can make a **side-by-side comparison of Docker vs CRI-O vs containerd in OpenShift** so it’s very clear why CRI-O is preferred. Do you want me to do that?


