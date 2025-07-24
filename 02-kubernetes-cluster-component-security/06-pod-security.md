# Pod Security

- Pod Security is crucial in securing workloads in a Kubernetes cluster. 
- It helps enforce how pods should be configured and run within the cluster to reduce the attack surface and prevent privilege escalation.

## 🚨 Why Pod Security?
- Kubernetes allows pods to run with high privileges by default.
- Misconfigured pods can lead to:
  - Host access
  - Privilege escalation
  - Access to sensitive resources

## Evolution of Pod Security in Kubernetes
- Kubernetes originally enforced Pod restrictions through **Pod Security Policies (PSP)**. 
- As of v1.25, PSP was removed in favor of the simpler, namespace-scoped **Pod Security Admission (PSA)** and **Pod Security Standards (PSS)**, which are now stable.

---

## Pod Security Policies (PSP)
- PSP was a cluster-level admission controller that validated each Pod creation request against defined policy objects. 
- If a request violated any rule, it was rejected.

### Enabling the PSP Admission Controller
- On your API server, include `PodSecurityPolicy` in the admission plugins:
    ```ini
    ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=${INTERNAL_IP} \
  --allow-privileged=true \
  --authorization-mode=Node,RBAC \
  --enable-admission-plugins=PodSecurityPolicy \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  # …other flags…
  ```

### Defining a Basic PSP
- A minimal PSP that disallows privileged containers:
    ```yml
    apiVersion: policy/v1beta1
    kind: PodSecurityPolicy
    metadata:
    name: example-psp
    spec:
    privileged: false
    seLinux:
        rule: RunAsAny
    supplementalGroups:
        rule: RunAsAny
    runAsUser:
        rule: RunAsAny
    fsGroup:
        rule: RunAsAny
    volumes:
        - '*'
    ```

### Creating a More Restrictive PSP
- Tighten policies to enforce non-root users, drop capabilities, and restrict volumes:
    ```yml
    apiVersion: policy/v1beta1
    kind: PodSecurityPolicy
    metadata:
    name: example-psp-restrictive
    spec:
    privileged: false
    seLinux:
        rule: RunAsAny
    supplementalGroups:
        rule: RunAsAny
    runAsUser:
        rule: MustRunAsNonRoot
    requiredDropCapabilities:
        - CAP_SYS_BOOT
    defaultAddCapabilities:
        - CAP_SYS_TIME
    volumes:
        - persistentVolumeClaim
    ```

- Key restrictions:
    - `requiredDropCapabilities` removes dangerous capabilities by default.
    - `defaultAddCapabilities` lets you explicitly add safe capabilities.
    - Volume types are limited to `persistentVolumeClaim`-no host-mounted paths.

---

## Granting PSP Access via RBAC
- Even with PSP enabled, you must grant subjects permission to use a specific PSP object.

### Role for PSP Usage
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: psp-example-role
rules:
- apiGroups: ["policy"]
  resources: ["podsecuritypolicies"]
  resourceNames: ["example-psp"]
  verbs: ["use"]
```

### RoleBinding to a ServiceAccount
```yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: psp-example-rolebinding
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: psp-example-role
```

> [!WARNING]
> Without these RBAC bindings, all Pod creation requests will be denied once the PSP admission plugin is enabled.

---

## Common Challenges with PSP
- Not enabled by default—manual API server configuration is required.
- Complex policy rollout—existing clusters must define PSPs for every use case.
- RBAC overhead—each user or service account needs separate Role and RoleBinding.
- Controller workloads (Deployments, DaemonSets) also require PSP access.
- PSP could mutate Pod specs (e.g., add default capabilities), a feature not carried over to PSA.

## Transition to Pod Security Admission (PSA)
- Pod Security Admission and Pod Security Standards simplify namespace-level enforcement without PSP’s RBAC complexity or mutating behavior.

| Aspect               | Pod Security Policy (PSP)       | Pod Security Admission (PSA)        |
|----------------------|---------------------------------|--------------|
| Scope                | Cluster-wide                    | Namespace |
| Lifecycle            | Deprecated/Removed (v1.25)      | Stable since v1.25 |
| RBAC Complexity      | High (Roles & Bindings per PSP) | Lower (Namespace annotations) |
| Mutating Capability  | Yes                             | No                               |
| Configuration        | Custom CRDs                     | Built-in enforce, audit, warn modes |