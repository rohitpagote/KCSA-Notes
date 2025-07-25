# Image Repository Security

- Image repositories (registries) are the source of container images. 
- Securing these repositories is crucial to prevent the introduction of malicious or compromised images into Kubernetes clusters.

## Understanding Image Names
- Docker interprets `image: nginx` as `library/nginx` under the hood. 
- The full naming convention is:
    ```ini
    [registry]/[user-or-namespace]/[repository]:[tag]
    ```
    - Omit the registry → defaults to Docker Hub (`docker.io`)
    - Omit the namespace → defaults to `library` (the official account)
- Specifying:
    ```ini 
    image: library/nginx
    ```
  is equivalent to:
    ```ini
    image: docker.io/library/nginx
    ```
- We can also pull from other public registries. For example, Google’s registry hosts Kubernetes test images:
    ```ini
    image: gcr.io/kubernetes-e2e-test-images/dnsutils
    ```

### Common Public Registries

| Registry | URL | Use Case |
|----------|-----|----------|
| Docker Hub | docker.io | Default public images |
| Google Artifact Registry | gcr.io | Google-hosted Kubernetes images |
| Quay.io | quay.io | CI/CD and enterprise images |

### Using a Private Registry

| Provider | Link |
|----------|------|
| AWS ECR   | https://aws.amazon.com/ecr/ |
| Azure Container Registry | https://azure.microsoft.com/services/container-registry/ |
| Google Artifact Registry | https://cloud.google.com/artifact-registry | 

To pull from a private registry, follow these steps:

#### 1. Authenticate locally (for pushing and testing)
```bash
docker login private-registry.io
# Username: registry-user
# Password: ********
# WARNING! Your password will be stored unencrypted in ~/.docker/config.json.
# Login Succeeded
```
 > [!NOTE]
 > Avoid committing `~/.docker/config.json` to version control. Store credentials securely (e.g., using a secrets manager).

#### 2. Create a Kubernetes Secret of type `docker-registry` so worker nodes can pull the image:
```bash
kubectl create secret docker-registry regcred \
  --docker-server=private-registry.io \
  --docker-username=registry-user \
  --docker-password=registry-password \
  --docker-email=registry-user@org.com
```

#### 3.Reference the Secret in your Pod spec under imagePullSecrets:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: internal-app-pod
spec:
  containers:
    - name: internal-app
      image: private-registry.io/apps/internal-app
  imagePullSecrets:
    - name: regcred
```

---

## 🧱 Key Security Concerns

| Concern | Description |
|--------|-------------|
| **Unauthorized Access** | Anyone can pull/push images if access is not restricted. |
| **Image Tampering** | Images could be altered if not signed or verified. |
| **Vulnerability Injection** | Insecure registries may host outdated or vulnerable images. |

---

## 🔐 Securing the Image Registry

### 1. **Use Private Registries**
- Prevents public access to sensitive images.
- Examples: Amazon ECR, Google Container Registry (GCR), Azure ACR, Harbor.

### 2. **Enforce Authentication and Authorization**
- Require credentials (tokens, passwords, certificates).
- Define roles:
  - **Reader** – Can pull images.
  - **Writer** – Can push new versions.
  - **Admin** – Full control.

### 3. **Enable TLS**
- Use HTTPS for secure data transfer.
- Prevents man-in-the-middle attacks when pulling images.

### 4. **Enable Image Signing & Verification**
- Tools like **Cosign**, **Notary** (used with Docker Content Trust).
- Kubernetes can be configured to **only allow signed images**.
