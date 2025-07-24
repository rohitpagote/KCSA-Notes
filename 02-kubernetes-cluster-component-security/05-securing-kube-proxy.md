# Securing Kube Proxy

- `kube-proxy` is a Kubernetes network component that manages network rules on nodes. 
- It enables communication between services and pods across the cluster by managing IP tables or IPVS rules.

## 🔐 Security Concerns
- Runs with elevated privileges.
- Interacts directly with network stack.
- Misconfigurations can allow lateral movement or DoS attacks.

## 🔧 Securing kube-proxy

### 1. Locate the Kube-Proxy Process and Configuration
- Identify the running kube-proxy and its config file:
    ```bash
    $ ps -ef | grep kube-proxy
    root      5351  5134  0 04:22 ?        00:00:04 /usr/local/bin/kube-proxy \
        --config=/var/lib/kube-proxy/config.conf \
        --hostname-override=controlplane --color=auto kube-proxy
    ```

- The `--config` flag points to the primary configuration:
    ```yml
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    bindAddress: 0.0.0.0
    bindAddressHardFail: false
    clientConnection:
    acceptContentTypes: ""
    burst: 0
    contentType: ""
    kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
    qps: 0
    clusterCIDR: 172.17.0.0/16
    ```

- The kubeconfig entry specifies where kube-proxy retrieves its API credentials.

### 2. Secure the kubeconfig File
- Protecting the kubeconfig file prevents unauthorized access to the API server.

#### 2.1 Verify Permissions and Ownership
- Use a table to validate file permissions and ownership:

| File                                    | Permissions  | Owner        |
|-----------------------------------------|--------------|--------------|
| `/var/lib/kube-proxy/config.conf`       | 644          | root:root |
| `/var/lib/kube-proxy/kubeconfig.conf`   | 600 (or 644) | root:root |

```bash
# Check permission
stat -c %a /var/lib/kube-proxy/kubeconfig.conf
# Check owner and group
stat -c %U:%G /var/lib/kube-proxy/kubeconfig.conf
```

### 3. Enforce TLS Encryption for API Connectivity
- Open the kubeconfig to confirm TLS settings and service-account authentication:
    ```yml
    apiVersion: v1
    kind: Config
    clusters:
    - cluster:
        certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        server: https://controlplane:6443
    name: default
    contexts:
    - context:
        cluster: default
        namespace: default
        user: default
    name: default
    current-context: default
    users:
    - name: default
    user:
        tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    ```
- `certificate-authority` ensures the API server’s certificate is validated.
- `server: https://...` enforces encrypted HTTPS connections.
- Service-account `tokenFile` grants authenticated, RBAC-controlled access.

## 4. Enable Audit Logging
Audit logs help you monitor all kube-proxy actions and detect suspicious activity.

1. Create an audit policy (e.g., /etc/kubernetes/audit-policy.yaml):
    ```yml
    apiVersion: audit.k8s.io/v1
    kind: Policy
    rules:
    # Log all actions by kube-proxy
    - level: Metadata
        users: ["system:kube-proxy"]

    # Optionally monitor changes to core resources
    - level: Metadata
        resources:
        - group: ""
            resources: ["pods", "services", "endpoints"]

    # Skip logging for all other requests
    - level: None
    ```

2. tream audit events to verify logs:
    ```bash
    tail -f /var/log/audit/audit.log | jq .objectRef.resource
    ```

---

# ✅ Summary
- Limit RBAC permissions for kube-proxy.
- Enable client certificate authentication.
- Use hardened images and run with minimal privileges.
- Use read-only root filesystems.
- Restrict network access to kube-proxy ports.
- Monitor logs for unusual activity.
