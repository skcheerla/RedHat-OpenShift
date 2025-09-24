Connecting OpenShift with Dell PowerMax storage is primarily done using the **Container Storage Interface (CSI) driver** for Dell PowerMax. This driver allows OpenShift to dynamically provision persistent storage volumes from the PowerMax array. The recommended and most straightforward method is to use the **Dell Container Storage Modules (CSM) Operator** available in the OpenShift OperatorHub.

The following is a high-level, step-by-step procedure. Please note that this process assumes you have a configured Dell PowerMax array with Unisphere, and the OpenShift cluster nodes have the necessary network connectivity (iSCSI or Fibre Channel) to the storage array.

### Step 1: Install Dell CSM Operator

The Dell CSM Operator is the most direct way to install and manage all of Dell's CSI drivers, including the one for PowerMax. It automates much of the deployment and lifecycle management.

1.  **Access the OpenShift OperatorHub**: Log in to the OpenShift web console as an administrator.
2.  Navigate to **Operators \> OperatorHub**.
3.  Search for **"Dell Container Storage Modules Operator"** and click on it.
4.  Click **"Install"** and choose your desired installation options, such as the namespace (it's often recommended to install it in a dedicated namespace like `dell-storage-operators`).
5.  Click **"Install"** to deploy the operator.

### Step 2: Configure Prerequisites

Before you can create the CSI driver instance, you need to prepare the OpenShift cluster and provide the necessary credentials for the PowerMax array.

1.  **Create a dedicated namespace**: Create a new project or namespace for your PowerMax storage driver.
    ```bash
    oc new-project dell-storage-powermax
    ```
2.  **Create a Kubernetes Secret for credentials**: The CSI driver needs credentials to communicate with the PowerMax Unisphere management endpoint.
      * Encode your Unisphere username and password in base64.
        ```bash
        echo -n 'your_unisphere_username' | base64
        echo -n 'your_unisphere_password' | base64
        ```
      * Create a YAML file (e.g., `powermax-creds.yaml`) for the secret, populating it with the base64-encoded values.
    <!-- end list -->
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: powermax-creds
      namespace: dell-storage-powermax
    type: Opaque
    data:
      username: <base64_username>
      password: <base64_password>
    ```
      * Apply the secret to the cluster.
        ```bash
        oc apply -f powermax-creds.yaml
        ```

### Step 3: Deploy the CSI PowerMax Driver

Now that the operator is running and the credentials are in place, you can create a custom resource (CR) to deploy the PowerMax CSI driver.

1.  **Create the CSI Driver CR**: Navigate to the **"Installed Operators"** section in the OpenShift console and select the Dell CSM Operator.
2.  Click on the **"Dell CSI PowerMax Driver"** API and then **"Create CSIPowerMax"**.
3.  You will be presented with a YAML editor. Provide the necessary details for your PowerMax array:
      * `unisphereHost`: The IP address of your Unisphere management endpoint.
      * `srpId`: The ID of the Storage Resource Pool (SRP) you want to use for dynamic provisioning.
      * `symid`: The System Identifier (SymID) of your PowerMax array.
      * `storageProtocol`: The protocol for connectivity (e.g., `FC` or `iSCSI`).
4.  Apply the YAML file. This will trigger the Dell CSM Operator to deploy the PowerMax controller and node pods.

### Step 4: Create a StorageClass

Once the CSI driver is successfully deployed, you can create a **StorageClass** that OpenShift users can reference to provision storage dynamically.

1.  Create a YAML file for the StorageClass (e.g., `powermax-sc.yaml`).
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: powermax-gold
    provisioner: csi-powermax.dellemc.com
    parameters:
      SRP: "SRP_1"
      SYMID: "000497600159"
      ServiceLevel: "Gold"
      csi.storage.k8s.io/fstype: "xfs"
    reclaimPolicy: Delete
    volumeBindingMode: WaitForFirstConsumer
    ```
2.  Apply the StorageClass to your cluster.
    ```bash
    oc apply -f powermax-sc.yaml
    ```

### Step 5: Provision and Use Persistent Volumes

Finally, developers can now create **Persistent Volume Claims (PVCs)** that reference the `powermax-gold` StorageClass. This will automatically trigger the CSI driver to provision a new volume on the Dell PowerMax array and bind it to the PVC.

1.  Create a PVC manifest (e.g., `pvc.yaml`).
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: my-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
      storageClassName: powermax-gold
    ```
2.  Apply the PVC.
    ```bash
    oc create -f pvc.yaml
    ```

After the PVC is created, you can mount it into your application's pod.

This video demonstrates a very similar process for provisioning storage using the Dell PowerStore CSI driver on OpenShift Virtualization.

[Dell PowerStore Enterprise Data Services with OpenShift Virtualization](https://www.youtube.com/watch?v=cQxn8Ao2zyo)
http://googleusercontent.com/youtube_content/0
