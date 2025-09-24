Using AWS, the step-by-step procedure for each scenario leverages **Elastic Block Store (EBS)** for block storage, **Elastic File System (EFS)** for file storage, and **Simple Storage Service (S3)** for object storage, with OpenShift's native capabilities to provision and manage them.

-----

### 1\. Database in a Pod (EBS) 💾

EBS volumes are a perfect match for a database because they are high-performance, block-level storage that can be attached to a single EC2 instance at a time, which aligns with the `ReadWriteOnce` access mode required by most databases.

**Procedure:**

1.  **StorageClass Creation:** An administrator defines a **StorageClass** that uses the AWS EBS CSI driver to dynamically provision volumes.
      * The `provisioner` is set to `ebs.csi.aws.com`.
      * Parameters like `type` (`gp3` for general purpose SSDs or `io1` for provisioned IOPS) and `fsType` (`ext4` or `xfs`) are specified.
      * This object can be created via the OpenShift console or `oc` CLI.
    <!-- end list -->
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: gp3-sc
    provisioner: ebs.csi.aws.com
    parameters:
      type: gp3
      fsType: xfs
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    ```
2.  **Persistent Volume Claim (PVC) Creation:** The developer creates a PVC in their application's manifest file.
      * The `storageClassName` must match the StorageClass created in the previous step.
      * They specify the required `accessModes` (`ReadWriteOnce` for a database) and `resources.requests.storage`.
    <!-- end list -->
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: postgres-data-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 50Gi
      storageClassName: gp3-sc
    ```
3.  **Pod Creation:** The PostgreSQL pod's manifest is configured to reference the PVC. When the pod is created, OpenShift's control plane dynamically provisions an EBS volume on AWS, attaches it to the EC2 instance where the pod is scheduled, and mounts it to the specified path in the container.
      * The `volumes` section references the PVC.
      * The `volumeMounts` section mounts the volume into the container's file system.
    <!-- end list -->
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: postgres-pod
    spec:
      containers:
        - name: postgres-container
          image: postgres:13
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-data-pvc
    ```

-----

### 2\. CI/CD Pipeline Artifacts (EFS) 📦

EFS is a fully managed NFS service on AWS, which is essential for CI/CD pipelines as it allows multiple pipeline stages (pods) to concurrently read and write to the same shared file system using the **ReadWriteMany** access mode.

**Procedure:**

1.  **EFS Provisioning:** An administrator manually creates an EFS file system on AWS and configures it for the OpenShift cluster, ensuring the appropriate mount targets and security groups are in place.
2.  **Persistent Volume (PV) Creation:** Unlike the dynamic provisioning in the first scenario, a PV for EFS is typically created manually by the administrator because EFS is a pre-existing resource. This PV points to the EFS file system.
      * `oc create` is used to create the PV from a YAML file.
      * The `accessModes` are set to `ReadWriteMany`.
      * The `nfs` section specifies the EFS server IP and path.
    <!-- end list -->
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: efs-pv
    spec:
      capacity:
        storage: 100Gi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Retain
      nfs:
        path: /
        server: <EFS_SERVER_IP>
      mountOptions:
        - vers=4.1
        - rsize=1048576
        - wsize=1048576
        - hard
        - timeo=600
        - retrans=2
    ```
3.  **Persistent Volume Claim (PVC) Creation:** The CI/CD pipeline's pods create a PVC that requests `ReadWriteMany` access and the required storage capacity. OpenShift's control plane will automatically bind this PVC to the available EFS-backed PV.
4.  **Pod Integration:** All pods within the pipeline (e.g., build pod, test pod) reference the same PVC, allowing them to share the EFS volume.

-----

