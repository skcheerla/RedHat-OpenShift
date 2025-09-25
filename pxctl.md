*****************************************************************

📄 Step 1: Create portworx-share StorageClass YAML

Create a file called portworx-share.yaml:

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: portworx-share
provisioner: pxd.portworx.com
parameters:
  sharedv4: "true"
  repl: "2"              # Optional: adjust replication factor
  fs: "ext4"             # Or "xfs"
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: Immediate

📥 Step 2: Apply the StorageClass in OpenShift
oc apply -f portworx-share.yaml


Verify:

oc get storageclass portworx-share

oc get storageclass portworx-share
NAME             PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
portworx-share   pxd.portworx.com   Delete          Immediate           true                   2m3s



📦 Step 3: Create a PVC Using portworx-share

Save this as shared-pvc.yaml:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName:




oc apply -f shared-pvc.yaml


# oc new-app --name=balanewapp1 --image=image-registry.openshift-image-registry.svc:5000/testing-oc-jsr/my-nginx-jsr
warning: Cannot find git. Ensure that it is installed and in your path. Git is required to work with git repositories.
--> Found container image 8f7427a (3 weeks old) from image-registry.openshift-image-registry.svc:5000 for "image-registry.openshift-image-registry.svc:5000/testing-oc-jsr/my-nginx-jsr"

    * An image stream tag will be created as "balanewapp1:latest" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "balanewapp1" created
    deployment.apps "balanewapp1" created
    service "balanewapp1" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/balanewapp1'
    Run 'oc status' to view your app.

 
 #oc set volume deployment/balanewapp1 \
  --add \
  --name=shared-vol \
  --claim-name=shared-pvc \
  --mount-path=/usr/share/nginx/html
deployment.apps/balanewapp1 volume updated


To verify

to getinto ruinning pod
#oc get pods
#oc rsh balanewapp1-7c7d6b9774-mm6pv


Create Content in the Shared Volume

Once inside the container:

cd /usr/share/nginx/html
echo "<h1>Hello from shared PVC!</h1>" > index.html


oc expose service balanewapp1

oc get route

http://balanewapp1-testing-oc-jsr.apps.ocp.yourdomain.com


you should see 
Hello from shared PVC!



Test Shared Volume Behavior (Optional)

You can scale the deployment to multiple pods:

#oc scale deployment balanewapp1 --replicas=2


Then rsh into the second pod, check the same /usr/share/nginx/html directory — you’ll see the same index.html file. This proves the Portworx RWX shared volume is working correctly.






****************************************************************


To find a Portworx daemonset pod:

oc -n portworx get pods -l name=portworx

login to the pod like below

oc exec -it -n portworx px-cluster-38bfc7c4-8hjjhvbhjvjhvjhvjhv-bq2fl -- /bin/sh

sh-5.1# pxctl status
Status: PX is operational

#pxctl service pool show

 Size: 2.0 TiB
        Status: Online
        Has metadata:  Yes
        Drives:
        1: /dev/mapper/mpathbj, Total size 2.0 TiB, Online
        Cache Drives:
        No Cache drives found in this pool



        StorageClasses Are Just Policy Templates

Kubernetes StorageClasses in Portworx don’t define storage capacity limits. They define:

Replication factors

Encryption settings

Snapshot policies

Shared volume support

Volume size settings (defaults)

File system type (fs)

IO profile (io_profile)

etc.

They do not carve out dedicated quotas or slices of total storage.



📌 Example:

If a Portworx node has 1 TB of available storage, then:

Volumes created via px-csi-db

Volumes created via px-csi-replicated

Volumes created via px-csi-db-encrypted

…will all consume storage from that same 1 TB pool — based on the volume size requested by the PVC.




























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
