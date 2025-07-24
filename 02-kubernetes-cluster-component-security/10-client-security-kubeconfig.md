# Client Security kubeconfig
 
- The `kubeconfig` file contains credentials and cluster access configurations for Kubernetes users. 
- Securing this file is critical to prevent unauthorized access to the cluster.

## 📂 What is `kubeconfig`?
- A YAML file used by `kubectl` to authenticate and interact with a Kubernetes cluster.
- Default location: `~/.kube/config`
- Contains:
  - **Clusters**: API server URLs and certificates
  - **Users**: Authentication details like tokens or client certs
  - **Contexts**: Maps/binds users to clusters

## 🚨 Why is `kubeconfig` Sensitive?
- Holds **authentication credentials** including tokens, certificates, or passwords.
- Anyone with access to it can impersonate the user and access the cluster.

---

## Kubeconfig 

### 1. Direct API Access with curl
- If your API server is at my-kube-playground:6443 and you’ve generated:
    CA certificate: `ca.crt`
    Client certificate: `admin.crt`
    Client key: `admin.key`
- You can query the API directly:
    ```bash
    curl https://my-kube-playground:6443/api/v1/pods \
    --key admin.key \
    --cert admin.crt \
    --cacert ca.crt

    {
        "kind": "PodList",
        "apiVersion": "v1",
        "metadata": {
            "selfLink": "/api/v1/pods"
        },
        "items": []
    }
    ```

### 2. Equivalent kubectl Commands
- With kubectl, the same request requires these flags:
    ```bash
    $ kubectl get pods \
    --server https://my-kube-playground:6443 \
    --client-key admin.key \
    --client-certificate admin.crt \
    --certificate-authority ca.crt
    ```
- Typing long flag lists is tedious. Let’s streamline this with a kubeconfig file.

> [!NOTE]
> By default, `kubectl` looks for` ~/.kube/config`. We can override with `--kubeconfig=PATH`.

### 3. Creating and Using a kubeconfig File
- Save your cluster, user, and context definitions in a YAML file (e.g., `config`), then run:
    ```bash
    kubectl get pods --kubeconfig=config
    # Or, if you place it at ~/.kube/config:
    kubectl get pods
    ```

### 4. Structure of a kubeconfig File

| Section  | Description                                                    |
|----------|----------------------------------------------------------------|
| Clusters | Definitions of Kubernetes clusters (name, server URL, CA info) | 
| Users    | Credentials (client cert/key or token) for each user           | 
| Contexts | Mappings of user <-> cluster, with optional namespace setting  |

```yml
apiVersion: v1
kind: Config

clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: ca.crt

users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key

contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin

current-context: my-kube-admin@my-kube-playground
```

### 5. Viewing and Switching Contexts
- Use built-in commands to inspect or switch contexts:

| Command                                           | Description                               |
|---------------------------------------------------|-------------------------------------------|
| `kubectl config view`                             | Show merged config (defaults + overrides) | 
| `kubectl --kubeconfig=my-config config view`      | View a specific file  | 
| `kubectl config use-context prod-user@production` | Switch current context to production  |

### 6. Setting a Default Namespace
- Embed `namespace` in the context to avoid adding `-n` on every command:
    ```yml
    contexts:
    - name: admin@production
    context:
        cluster: production
        user: admin
        namespace: finance
    ```

### 7. Embedding Certificate Data Directly
- Inline your certs and keys as base64 to avoid external files:
    ```yml
    clusters:
    - name: production
    cluster:
        server: https://172.17.0.51:6443
        certificate-authority-data: |
        LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXREND...
    users:
    - name: admin
    user:
        client-certificate-data: |
        LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJXREND...
        client-key-data: |
        LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQo...
    ```
- Generate and inspect base64 fields:
    ```bash
    # Encode files
    cat ca.crt      | base64
    cat admin.crt   | base64
    cat admin.key   | base64

    # Decode embedded data
    echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0t" | base64 --decode
    ```

---

## 🛡️ Security Best Practices

### 1. 🔒 File Permissions
- Ensure the `kubeconfig` file is readable only by the owner:
    ```bash
    chmod 600 ~/.kube/config
    ```

### 2. 👤 User Isolation
- Each user should have their own `kubeconfig` file.
- Avoid sharing kubeconfig files across users.

### 3. 🔁 Token Expiry and Rotation
- Use short-lived tokens or client certificates with expiration.
- Rotate credentials periodically.

### 4. 🔍 Auditing
- Enable audit logging on the API server to track actions by users.
- Helps detect if a stolen kubeconfig is being misused.

### 5. 📁 Store Securely
- Do not check kubeconfig into version control.
- Use encrypted secrets storage if you need to store it programmatically.
