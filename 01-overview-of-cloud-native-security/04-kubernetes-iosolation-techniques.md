# Kubernetes Isolation Techniques

Isolation ensures multi-tenancy safety, fault containment, and tenant workload segregation.

## 1. Namespace Separation
- Isolate teams/projects using **separate namespaces**: `prod`, `dev`, `team-a`, etc.
- Reduces **blast radius** (potential impact or scope of damage that a faulty deployment, misconfiguration, or security breach could have on a system or application.) and simplifies resource management.
- Use clear naming conventions (e.g., `team-a`, `team-b`). 

## 2. Network Policies
- By default, Pods can communicate across namespaces.
- Use `NetworkPolicy` to restrict ingress/egress.
- Example: Allow only Pods in the prod namespace to receive ingress traffic from peers within prod:

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-internal-prod-namespace
  namespace: prod
spec:
  podSelector: {}        # Select all Pods in prod
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector: {}  # Only Pods in the same namespace
```

## 3. Role-Based Access Control (RBAC)
- Enforces **least privilege access** across namespaces.

| Role         | Namespace   | Permission                   |
|--------------|-------------|------------------------------|
| `dev-reader` | prod        | `get`, `list`                |
| `dev-admin`  | dev         | `create`, `delete`, `update` |

- Apply Roles and RoleBindings per namespace.

## 4. Resource Quotas and Limits
- `ResourceQuotas` control overall resource consumption per namespace. 
- Pod-level resource requests and limits prevent individual workloads from exhausting CPU or memory.

```yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
```

## 5. Security Context
- By default, containers may run as root, which heightens risk if compromised. 
- Use a `securityContext` to enforce non-root execution and restrict privileges.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  namespace: dev
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: backend-container
      image: nginx:latest
      securityContext:
        allowPrivilegeEscalation: false
```

---

# ✅ Summary
1. **Namespace Separation:** Logical multi-tenant isolation
2. **Network Policies:** Control pod-to-pod traffic
3. **RBAC:** Enforce least-privilege permissions
4. **Quotas & Limits:**	Prevent resource monopolization
5. **Security Contexts:** Enforce non-root and restrict capabilities
