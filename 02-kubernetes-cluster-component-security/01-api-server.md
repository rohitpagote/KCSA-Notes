# Kubernetes API Server Security

The API Server is the central point of control in a Kubernetes cluster — all actions go through it. It must be secured tightly.

## 1. Main Security Functions  
- **Authentication**: Verifies who the user is. 
- **Authorization**: Verifies what access the user is having.  
- **Encryption**: Secures communication between components.

### 🔐 Authentication Methods  
Choose based on cluster use:

| Type                 | Description                                  | Use Case                        |
|----------------------|----------------------------------------------|---------------------------------|
| Static Credentials   | Username/password in files                   | Test clusters                   |
| Bearer Tokens        | Secrets or service account tokens            | CI/CD, automation               |
| Client Certificates  | X.509 certificates for users and components  | Production environments         |
| External Providers   | LDAP, OIDC, webhook tokens                   | Enterprise SSO                  |
| Service Accounts     | Built-in for in-cluster pods                 | Pod API access (least privilege)|  

### 🛡️ Authorization Modes  
Controls what authenticated identities can do:

| Module              | Description                                  |
|---------------------|----------------------------------------------|
| RBAC                | Grant roles to users or service accounts     |
| ABAC                | Policy-based on user attributes              |
| Node Aythorization  | Restricts kubelet actions to pods running on the same node 
| Webhook Mode        | Delegates authorization to an external service | 

⚠️ Always apply least privilege in RBAC.

### 🔐 TLS Encryption  
- All communication between the API server and other cluster components is encrypted via TLS. This includes:
    - etcd cluster
    - kube-controller-manager
    - kube-scheduler
    - kubelet and kube-proxy on worker nodes

---

# ✅ Summary  
- Enable **strong authentication** (prefer certs or external).
- Implement **RBAC** with least privilege.
- Enforce **TLS encryption** for all communications.
- **Disable insecure API access channels**.
- Manage **certificates securely** and rotate regularly.
