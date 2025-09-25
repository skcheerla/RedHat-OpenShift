Good catch 👍 — you won’t see `/opt/pwx/bin/pxctl` on a normal OpenShift node or your workstation.
That binary lives **inside the Portworx container** (because Portworx runs as a DaemonSet on worker nodes).

Here’s how you can access it:

---

### 🔹 Step 1: Find a Portworx pod

Portworx pods usually run in the `kube-system` or `portworx` namespace. Run:

```bash
oc get pods -A | grep portworx
```

You should see something like:

```
kube-system   portworx-abc123   1/1   Running   10m
kube-system   portworx-def456   1/1   Running   10m
```

---

### 🔹 Step 2: Exec into a Portworx pod

Pick one pod and run:

```bash
oc exec -it -n kube-system portworx-abc123 -- /bin/sh
```

---

### 🔹 Step 3: Run `pxctl` inside the pod

Once inside, the `pxctl` binary is already on the path, so just run:

```bash
pxctl status
```

Example output:

```
Cluster ID: 1234abcd
Status: OK
Nodes: 3
   IP         ID      Used   Capacity   Status
   10.0.0.5   node1   200G   500G       Online
   10.0.0.6   node2   150G   500G       Online
   10.0.0.7   node3   100G   500G       Online

Cluster-wide Storage
   Total Capacity : 1500 GB
   Used           : 450 GB
   Available      : 1050 GB
```

---

✅ If you don’t want to exec into pods every time:
Portworx also provides a **kubectl/oc plugin** called `pxctl` (sometimes installed separately), but in most OpenShift clusters the standard way is **exec into a Portworx pod** and run it there.

---

Do you want me to also give you a **pure OpenShift way** (using `oc describe storagecluster` or `oc get pxstoragecluster` CRD) so you can check capacity without entering the pod?
