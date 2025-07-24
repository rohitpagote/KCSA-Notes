# Securing Container Runtime

The container runtime is responsible for running containers on the node. It plays a critical role in the Kubernetes architecture, making its security essential.

## 🧱 Common Container Runtimes
- **containerd**
- **CRI-O**
- **Docker** (deprecated for Kubernetes 1.24+)

## Why Migrate from Docker?
- Docker’s early releases suffered from a critical vulnerability that allowed containers to execute with root privileges on the host. 
- Modern container runtimes mitigate this risk by adhering to the **Container Runtime Interface (CRI)**.

## 🔐 Common Container Runtime Vulnerabilities
Several high-profile CVEs have underscored the need for a secure runtime:

| Vulnerability                | Impact             |
|------------------------------|---------------------------------|
| Dirty COW	                   | Linux kernel flaw allowing unauthorized root access        | 
| `runc` container breakout    | Overwriting the `runc` binary to gain host-level privileges | 
| Docker container escape      | Accessing sensitive host files from within a container  | 
| containerd denial of service | Forcing a host DoS via containerd resource exhaustion    | 
| containerd denial of service | Breaking out of CRI-O sandbox to the host   | 

## 🔧 Securing Container Runtime

### 1. **Use Minimal and Hardened Runtimes**
- Prefer **containerd** or **CRI-O** over Docker.
- These are designed for Kubernetes and have a smaller attack surface.

### 2. Least-Privilege Execution
- Avoid running containers as root. Assign non-root UIDs/GIDs to limit blast radius:
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
    name: least-privilege-pod
    spec:
    securityContext:
        runAsUser: 1000
        runAsGroup: 3000
    containers:
        - name: app
        image: my-app-image
    ```

### 3. Enforce a Read-Only Filesystem
- Prevent on-disk tampering by mounting the root filesystem as read-only:
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
    name: readonly-pod
    spec:
    containers:
        - name: app
        image: my-app-image
        securityContext:
            readOnlyRootFilesystem: true
    ```

### 4. Resource Limits
- Define CPU and memory limits to protect the node from denial-of-service attacks:
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
    name: resource-limits-pod
    spec:
    containers:
        - name: app
        image: my-app-image
        resources:
            limits:
            memory: "512Mi"
            cpu: "1"
    ```

### 5. Mandatory Access Control (SELinux & AppArmor)
#### SELinux
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
    name: selinux-demo
    spec:
    containers:
        - name: nginx
        image: nginx
        securityContext:
            seLinuxOptions:
            user: "system_u"
            role: "system_r"
            type: "spc_t"
            level: "s0:c123,c456"
    ```

#### AppArmor
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
    name: apparmor-demo
    annotations:
        container.apparmor.security.beta.Kubernetes.io/app: localhost/my-apparmor-profile
    spec:
    containers:
        - name: app
        image: my-app-image
    ```

### 6. Transition to containerd or CRI-O
- Docker support is deprecated in newer Kubernetes releases. 
- Migrate to containerd or CRI-O for enhanced security, performance, and forward compatibility.

### 7. Monitoring, Logging & Auditing
- Centralize logs and metrics to detect runtime anomalies:
    - Fluentd, Logstash, Elasticsearch for log aggregation
    - Prometheus & Grafana for metrics
    - Kubernetes Audit Logs for API event tracking

### 8. Drop Linux Capabilities
- Drop unnecessary Linux capabilities to reduce privilege.
    ```yml
    securityContext:
        capabilities:
            drop: ["ALL"]
    ```

### 9. Avoid Privileged Containers
- Do not use privileged: true unless absolutely necessary.
- It grants full access to the host.
