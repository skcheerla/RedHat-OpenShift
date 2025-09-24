Good one 👍 — you want to **allocate local storage (e.g. `/mnt/…` on your nodes)** for the **internal OpenShift image registry** pods.

In OpenShift 4.x, the **Image Registry Operator** manages the internal registry. By default, in new installs it often uses **`emptyDir`** (ephemeral storage) unless you configure **persistent storage**. If you want your registry pods to use a **local path like `/mnt`** on your worker nodes, you need to configure it explicitly.

---

## 🔹 1. Check Current Registry Configuration

```bash
oc get configs.imageregistry.operator.openshift.io/cluster -o yaml
```

Look for:

```yaml
storage:
  managementState: Managed
  emptyDir: {}
```

This means the registry is running with ephemeral storage (data lost on pod restart).

---

## 🔹 2. How to Use Local Storage for Registry

You **cannot directly mount `/mnt` with hostPath** into the registry (bad practice, unsupported for HA).
Instead, you provide a **PersistentVolume** backed by your `/mnt` directory, then configure the registry to use that PVC.

---

### Option A: Quick & Dirty — HostPath PV (for dev/test only ⚠️)

1. On a worker node, create the path:

   ```bash
   sudo mkdir -p /mnt/registry
   sudo chmod 777 /mnt/registry
   ```

2. Create a PV using hostPath:

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: registry-pv
   spec:
     capacity:
       storage: 100Gi
     accessModes:
       - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: registry-local
     hostPath:
       path: /mnt/registry
   ```

3. Create a PVC:

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: registry-pvc
     namespace: openshift-image-registry
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 100Gi
     storageClassName: registry-local
   ```

4. Patch the registry config to use this PVC:

   ```bash
   oc patch configs.imageregistry.operator.openshift.io/cluster \
     --type=merge -p '{"spec":{"managementState":"Managed","storage":{"pvc":{"claim":"registry-pvc"}}}}'
   ```

5. Check pods restart with PVC mounted:

   ```bash
   oc get pods -n openshift-image-registry
   oc describe pod -n openshift-image-registry <registry-pod> | grep Mount
   ```

---

### Option B: Recommended — Local Storage Operator (HA-capable)

Instead of hardcoding hostPath PVs, you use the **Local Storage Operator (LSO)** to manage `/mnt` or local disks on your worker nodes:

1. Install **Local Storage Operator** from OperatorHub.
2. Create a `LocalVolume` that points to `/mnt/registry` or a dedicated disk.

   ```yaml
   apiVersion: local.storage.openshift.io/v1
   kind: LocalVolume
   metadata:
     name: local-registry
     namespace: openshift-local-storage
   spec:
     storageClassDevices:
     - storageClassName: registry-local
       volumeMode: Filesystem
       fsType: xfs
       devicePaths:
       - /mnt/registry
   ```
3. This auto-creates PVs. Then create a PVC for the registry as in Step 3 above.
4. Patch the registry to use that PVC.

---

## 🔹 3. Verify

* Registry Config:

  ```bash
  oc get configs.imageregistry.operator.openshift.io/cluster -o yaml
  ```

  Should show:

  ```yaml
  storage:
    pvc:
      claim: registry-pvc
  ```
* Registry pod mounts:

  ```bash
  oc describe pod -n openshift-image-registry <registry-pod> | grep registry-pvc
  ```

---

## ✅ Summary

* The internal image registry defaults to **emptyDir (ephemeral)** if no storage is configured.
* To allocate **local `/mnt` storage**:

  1. Create a **PV** pointing to `/mnt` (via hostPath or Local Storage Operator).
  2. Create a **PVC** in `openshift-image-registry`.
  3. Patch the registry config to use that PVC.
* For **production**, Red Hat recommends using **Local Storage Operator** or cloud storage (NFS, ODF, EBS, etc.).

---

👉 Do you want me to give you a **ready-to-apply YAML bundle** (PV + PVC + Registry patch) for a quick dev/test setup using `/mnt/registry` on a worker node?
