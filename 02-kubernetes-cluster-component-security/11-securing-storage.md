# Securing Storage

- Securing storage in Kubernetes is essential because sensitive data (like credentials, secrets, configuration, etc.) is often persisted across Pods and containers. 
- Misconfigured storage can lead to data leakage or unauthorized access.

## 🔐 Key Security Risks in Kubernetes Storage
- **Unrestricted Access**: Improperly configured volumes may allow all Pods to access sensitive data.
- **Data Leakage**: Logs, backups, or shared storage might unintentionally expose secrets.
- **Lack of Encryption**: Data at rest and in transit must be encrypted to prevent interception or tampering.
- **Insecure Mounts**: HostPath and other volume types can expose critical parts of the host filesystem.

## 📁 Common Kubernetes Storage Types and Risks

| Storage Type   | Description                             | Security Risks                                      |
|----------------|-----------------------------------------|-----------------------------------------------------|
| **emptyDir**   | Temporary storage shared by containers  | Not encrypted, deleted when Pod terminates         |
| **hostPath**   | Mounts a host file/directory             | High risk of host compromise                       |
| **persistentVolume** (PV) | Long-term persistent storage    | Improper access control, data leakage               |
| **secret**, **configMap** | Store sensitive data as volumes | Base64 encoded only (not encrypted), misuse risk   |

## 🛡️ Best Practices for Securing Storage

### 1. 🔐 Use Encryption
- **Encrypt data at rest** via CSI drivers or cloud-native storage solutions
- Use **TLS** for storage systems that support encryption in transit


| Provider                     | Encryption Feature                                            | Reference | 
|------------------------------|---------------------------------------------------------------| ------------------| 
| AWS EBS                      | Customer-managed keys for EBS volumes                         | https://aws.amazon.com/ebs | 
| Azure Disk Storage           | Server-side encryption with platform or customer-managed keys | https://azure.microsoft.com/services/managed-disks/ | 
| Google Cloud Persistent Disk | CMEK/Customer-supplied encryption keys	                       | https://cloud.google.com/persistent-disk | 

- To enable encryption on AWS EBS via a custom StorageClass:
    ```yml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: encrypted-ebs
    provisioner: kubernetes.io/aws-ebs
    parameters:
        type: gp2
        encrypted: "true"
    ```

### 2. 🔒 Role-Based Access Control (RBAC) for Storage
- Restrict access to StorageClasses, PVs, and PVCs using Kubernetes RBAC. 
- Define granular roles and bind them to users or service accounts.

    ```yml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
        namespace: default
        name: pvc-reader
    rules:
        apiGroups: [""]
        resources: ["persistentvolumeclaims"]
        verbs: ["get", "list"]

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
        name: read-pvc-binding
        namespace: default
    subjects:
    - kind: User
        name: jane
        apiGroup: rbac.authorization.k8s.io
    roleRef:
        kind: Role
        name: pvc-reader
        apiGroup: rbac.authorization.k8s.io
    ```

### 3. 🔒 StorageClasses
- StorageClasses allows to standardize storage parameters—such as encryption, IOPS, and backup policies—across your cluster
    ```yml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: secure-storage
    provisioner: kubernetes.io/aws-ebs
    parameters:
        type: gp3
        encrypted: "true"
        iops: "3000"
    ```

### 4. 🔒 Use Pod Security Standards
- Enforce **PodSecurityPolicy** (deprecated) or **Pod Security Admission** to:
  - Disallow `hostPath` usage
  - Prevent privilege escalation

### 5. 📁 Restrict Volume Types
- Prefer **Persistent Volumes (PV)** with dynamic provisioning
- Avoid `hostPath`, especially for non-admin workloads

### 6. Backup and Disaster Recovery
- Implement automated backups and cross-cluster replication to guard against data loss, corruption, and ransomware.

| Tool      | Description | Link | 
|-----------|------------------------------------------------------------|--------------------------|
| Velero    | Open source backup, restore, and disaster recovery         | https://velero.io | 
| Portworx  | Enterprise-grade storage management and DR                 | https://portworx.com | 
| OpenEBS   | Containerized storage with snapshot and clone features     | https://openebs.io | 
| Kasten    | Policy-driven backup and mobility for Kubernetes volumes   | https://www.kasten.io | 

### 5. 🔍 Audit and Monitor
- Track storage metrics and access patterns to detect anomalies early. 
- Use Prometheus for data collection and Grafana for visualization.
- Important metrics:
    - Volume latency and throughput
    - PVC capacity versus usage
    - I/O error rates
    - Unauthorized mount or delete attempts

---

# ✅ Summary
- Encrypt data at rest and in transit
- Enforce RBAC for storage resources
- Standardize storage parameters with StorageClasses
- Automate backups and disaster recovery
- Monitor storage metrics and access patterns
