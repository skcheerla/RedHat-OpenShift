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

